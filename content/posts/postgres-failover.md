---
title: "Postgres Failover and Promotion"
date: 2021-07-16T15:19:44-04:00
draft: false
toc: false
images:
tags: 
  - postgres
---

I was on production support this week. Monday was quiet, but on Tuesday a variety of seemingly disconnected events came in: one app had no data for a certain metric, our Airflow instance was down, a pipeline reporting dashboard was down, and other pipelines were delayed. Eventually it became clear that these were all downstream symptoms of the real problem, which was that our primary PGDB instance had failed.

The obvious question is, why would you have an outage during a failover? The answer is that our Postgres deployment has serious room for improvement.

We have PGDB instances set up on two different hosts. One is a primary instance capable of taking reads and writes. The other is a secondary instance, capable of serving read requests only. Both are fronted by a load balancer which will direct you to the primary instance or the secondary instance if the primary is down.

There are a couple problems with our implementation that we should improve:

- We don’t have a true failover and promotion setup. If our primary goes down, the secondary can serve read requests but it will not be promoted to primary and start taking write requests. In order to allow writing to the DB, we have to get the primary back online. This means that even with a secondary DB instance, we are basically in an outage state until the primary can be fixed. __A better approach would be to have the secondary automatically promoted in the case of a failover situation and start taking write requests. Once back online, the former primary can catch up to the latest state via replication and becomes the new secondary.__

- We need two secondary DBs. If the primary goes out, we are down to a single secondary DB. If that host went down... Let’s not think about it. We have another host picked out for the purpose, we just need to set it up with PGDB and an SSL connection and get it replicating.

- Many systems still have hard-coded connections to the primary instance. These systems were completely down during the outage, even though we had a secondary DB instance. In the case of read-only systems, this is because the load balancer didn’t exist when the systems were created and the systems were never updated afterward. For read-write systems, this is because of the previous bullet point: only the primary instance can ever support write requests, so there’s no point in connecting the systems through the load balancer. __If we implement true failover and promotion, all systems can connect to our load balancer and be confident that a highly-available DB will be able to serve both read and write requests.__

Back to the outage. It was caused when an external team applied patches to our hosts, including the host that runs our primary instance. This host did not come back online successfully due to an issue with the new kernel. The external team filed a low-priority ticket (!) and moved on.

The next day, when we figured out what had happened, we were not able to restart the machine. We had to work with another team to get someone to go to the actual data center and downgrade the host to a previous kernel. Then the system came up and the PGDB instance came back online.

We were then able to start getting our platform back on its feet.