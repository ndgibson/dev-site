---
title: "Postgres: Dropping Multiple Partitions"
date: 2021-09-24T16:30:03-04:00
draft: false
toc: false
images:
tags: 
  - postgres
  - sql
---

Postgres partition support is powerful but not especially user-friendly. You may think (like I did) that partition support means all kinds of graceful first-class handling of partitions, but it really doesn't. Seemingly trivial things like "drop all partitions that meet a certain condition" aren't especially intuitive.

This week I was building an Airflow-orchestrated ETL that would take data every day and dump it into Postgres. I wanted to partition the Postgres table by date so that the table would perform well even after a year or two of daily runs. On the other hand, we don't need this data forever -  it doesn't make sense to keep around hundreds of old partitions. 

So our daily pipeline needs to clean up partitions older than a certain date (based on our retention policy) in addition to loading in the new daily data. The direct and simple approach would be to just drop the specific partition that corresponds to `today - MAX_RETENTION`, but what if the pipeline run for a certain day fails? Then that old partition will always be left hanging around, because the next pipeline run will be `tomorrow - MAX_RETENTION`.

What we really need to do is drop all partitions that match a certain set of criteria.

Here's the table:

```
CREATE TABLE IF NOT EXISTS foo (
  columnA varchar,
  run_date int
) PARTITION BY LIST (run_date);
```

The partition is on `run_date`, which would have values like `20210922`. It's a daywise partition so we don't need any more accuracy than that.

Then we manually create a partition for today's data:

```
CREATE TABLE foo_20210924
  PARTITION OF foo
  FOR VALUES IN (20210924);
```

And of course the `20210924` value is provided by Airflow as a template variable so that we can reuse these commands every day.

But once we've loaded our data, how do we drop partitions older than `RETENTION_MAX`, say 360 days ago or `20200924`? Here's what I ended up with:

```
DO
$do$
  BEGIN
    EXECUTE (
      SELECT
        coalesce(
          'DROP TABLE ' || string_agg(format('%I.%I', schemaname, tablename), ', '),
          'SELECT NULL'
        )
      FROM pg_catalog.pg_tables t
      WHERE schemaname = 'your_schema'
        AND tablename LIKE 'foo_%'
        AND REPLACE(tablename, 'foo_', '')::int < 20200924
    );
  END
$do$;
```

Whoa, what is this doing? Well, starting from the inside...

- Get us all the tables in the relevant schema that have a name like `foo_`. When we get this list, limit it to just the tables that (when you remove the `foo_` part) have a number that's less than our max retention date of `20200924`.
- Once we have that list, create a string that starts with `DROP TABLE` and ends with a comma-separated list of all the table names that matched our query in the previous step.
- If we didn't find any matching tables, then just return `SELECT NULL` so that our next step doesn't complain about a bad instruction.
- Now we have either our `DROP TABLE` string or our `SELECT NULL` string. `EXECUTE` it like a SQL command! `EXECUTE` will throw an error if you try to use it on a blank string, so that's why we have `SELECT NULL` as a default value.

Pretty cool. This will do exactly what we want. It'll drop all partitions of our `foo` table with a `run_date` older than a certain value. And of course you could modify the `WHERE` to be any other kind of conditions.

Warning: this is a massive footgun because it will search all the tables the user has access to and automatically drop anything that matches! That's why it's really important to limit the schema, set up specific `WHERE` conditions, and do some testing. If you don't, you could potentially drop Postgres system schema!