---
title: "Hadoop Intermediate Output Compression"
date: 2021-08-12T15:06:59-04:00
draft: false
toc: false
images:
tags: 
  - hadoop
---

Our Production Support rotation frequently gets alerted because hosts serving as Hadoop cluster nodes run out of disk space. The root cause is that the hosts store intermediate output files during the map-reduce process, and for certain jobs these intermediate files can consume ~70GB per host. Disk space becomes critically low because we don't have huge volumes, slowing down the hosts for all tasks and preventing other jobs from executing successfully.

This is really inefficient.

With two lines of config, we enabled intermediate output compression on our Hadoop cluster and reduced disk space usage by up to 95%. This helped our customers by preventing Hadoop jobs from failing due to low disk space, and help our teammates by preventing Production Support from getting alerted for hosts with low disk space.

There were several possible approaches to solving this issue:

* Add more nodes to the Hadoop cluster relieve disk space pressure on all nodes.
    * This would take a lot of config work and messing around with hosts, and doesn’t address the root problem of inefficient configuration.
* Alter the Hadoop config so that hosts use a higher-capacity mounted volume.
    * This would be easier than the first option, but also doesn’t address the root problem of inefficient configuration.
* Enabling intermediate output compression.
    * This could be as easy as a two-line config change, and may make our jobs dramatically more efficient in disk space usage.

We took the third route: intermediate output compression.

## Hadoop Compression

There are several codec options for compression:

|Codec |File Extension |Splittable? |Compression Ratio |Speed |
|--- |--- |--- |--- |--- |
|Gzip |.gz |No |Medium |Medium |
|Bzip2 |.bz2 |Yes |High |Slow |
|Snappy |.snappy |No |Medium |Fast |
|LZO |.lzo |No, unless indexed |Medium |Fast |

They can be evaluated against each other in terms of:

* Compression ratio (how much disk space does the codec save versus raw text files?)
* Speed (how much CPU is needed to compute the compression, and how much more slowly does the job run?)
* Splittable (map-reduce on raw text output can be parallelized across nodes, does the codec support that?)

We chose the `Bzip2` codec. Although it doesn’t compress as much as `Gzip` and is slower than `Snappy`, it retains support for splitting which allows for parallel map-reduce across nodes.

Probably a better solution would be to use a binary container format like Avro with Snappy, which would give you good compression and fast speed and also be splittable, but the work required to implement that was beyond what we could commit to right now.

## Testing Plan

Before we make changes to the cluster, we want to take an individual job, enable intermediate output compression, and compare disk usage between compressed and uncompressed runs. We’re using a job for the experiment that operates on a large volume of data and typically blocks the cluster during runtime.

By default the pipeline does not use compression. To enable compression, we modify our job config:

```
...
mapreduce.map.output.compress=true
mapred.map.output.compress.codec=apache.hadoop.io.compress.BZip2Codec
...
```

Online you’ll see a lot of examples showing how to enable intermediate compression using different syntax from what you see above. That syntax online is probably MRv1, which looks slightly different and is considered deprecated in MRv2/YARN. If you want to know how to tell what map-reduce framework the cluster is using, you can go to the Hadoop config file and look for `mapreduce.framework.name`:

```
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
<source>mapred-site.xml</source>
</property>
```

Here we can see that we need to use the YARN syntax because our Hadoop cluster is configured to use YARN (a.k.a. MRv2) as the map-reduce framework.

## Validation Plan

In order to evaluate the disk space impact of this job with and without compression, we need to look at disk space metrics on the relevant hosts. We have a couple metrics for our hosts; bytes used or byte percentage used should work fine. But which hosts do we need to look at?

Your job will probably list a couple hosts explicitly. But these hosts may just be the ones running the [HiveServer2 (HS2) service](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Overview), which allows queries to run against Hive. Map-reduce jobs will execute once Hive has been queried; you can see these jobs in the Hadoop tracker dashboard. These map-reduce jobs can run on any nodes in the Hadoop cluster, and intermediate data (which is what we’re interested in, since it’s triggering our alerts) can be stored in any of the cluster nodes.

So, to summarize, although the hosts listed explicitly may indeed see their disk space usage increase during the job run, there are many other host nodes in the cluster that will also be impacted.

To get a comprehensive look at all the disk space usage across the cluster during the job execution, we need to get the list of all cluster nodes from the Hadoop dashboard administrative settings under Cluster -> Nodes.

Then you can set up your metric query to cover all the impacted hosts.

## Results

### Uncompressed

Without compression, the job takes about 15-20 minutes to run and disk space usage across the cluster increases by about 2 TBs.

For a single run, the cluster disk space usage peaked at 2.75 TB. On a single node, disk space usage increases by about 25% to almost 100% usage.

### Compressed

With compression, the job takes about the same amount of time, 15-20 minutes. Whatever we’re losing in CPU compute to calculate compression, we are making up for in having free disk space to prevent hosts from getting bogged down. Across the cluster, disk space usage only goes up by 110 GB, which is 95% more efficient!

The cluster peaks at 850 GB now, and on a single node the disk space usage only goes up by a couple percent.

## Applying the Changes

This is a change to the map-reduce process in Hadoop, so we want to make the changes in `mapred-site.xml`. The catch is that we have to change it on all the nodes in the Hadoop cluster, both the Resource Manager node and all the other normal nodes.

### Update the Resource Manager

Let’s start with the Resource Manager. You want to change this file:

```
$HADOOP_HOME/etc/hadoop/mapred-site.xml
```

Your change will look like this:

```
<property>
  <name>mapreduce.map.output.compress</name>
  <value>true</value>
</property>
<property>
  <name>mapred.map.output.compress.codec</name>
  <value>apache.hadoop.io.compress.BZip2Codec</value>
</property>
```

### Update the Other Nodes

Next, go ahead and apply your config changes to all the other nodes in the cluster. Hopefully you have some automated way of doing this, especially if your cluster is large!

### Restart Everything

Now we need to restart both the Resource Node and all the other nodes in order for everything to pick up the new config.

From the Resource Manager node, just run this command to restart Hadoop:

```
$ yarn-daemon.sh --config /usr/local/hadoop/etc/hadoop stop resourcemanager
$ yarn-daemon.sh --config /usr/local/hadoop/etc/hadoop start resourcemanager
```

We can do something similar for the other nodes...again, hope you have an automated way of doing this.

```
yarn-daemon.sh --config /usr/local/hadoop/etc/hadoop stop nodemanager
yarn-daemon.sh --config /usr/local/hadoop/etc/hadoop start nodemanager
```

### Are We Good?

One quick way to validate that all is well is to run a quick map-reduce query in hive. From a Hive node, type `hive` and run a query. Note that running a normal `select *` won’t actually activate a map-reduce, so make sure you do a `count`. Of course, any table will do.

Also, go to the Resource Manager dashboard to make sure all the expected nodes show up in the list and that jobs have started to come through the queue.