---
title: "Structured Logging"
date: 2021-08-19T15:38:37-04:00
draft: false
toc: false
images:
tags: 
  - logging
---

## Value Proposition

You’re on Production Support. It’s just getting to be start-of-workday on the West Coast when you start to get all these reports in the support channel about systems and applications having problems. Pipelines are behind, data tables are showing blank or incorrect data, some features of applications aren’t working at all.

Wouldn’t it be great to go to a single place, run a single command, and in seconds see all the log messages across all of our systems, services, applications, and pipelines that have a log level of `ERROR` and a timestamp within the last six hours? You could probably find root cause a lot faster, which would make our customers happy and reduce your stress.
 
We need a couple things in order to do this, though.

First, we need *centralized logging*. All our systems, services, applications, and pipelines need to be transmitting their logs to a single place. This might be [Splunk](https://www.splunk.com/) or [Datadog](https://www.datadoghq.com/), who knows.

Second, we need *structured logging*. Even with all our logs in one place, without structured logs we won’t be able to easily search across all of our logs for various criteria such as log level, timestamp, message contents, module or function location, stacktrace, etc.

So what is structured logging?

## Humans vs. Machines

Let’s look at a log excerpt from a typical Python application:

```
2021-08-19 15:09:18 INFO db_connector - get_db_connection: DB Client received
2021-08-19 15:09:18 ERROR db_connector - execute_query: Error while querying the database: SSL SYSCALL error: EOF detected

2021-08-19 15:09:18 INFO db_connector - close_db_connection: Closing cursor
2021-08-19 15:09:18 INFO db_connector - close_db_connection: Placing connection back in pool
2021-08-19 15:09:18 ERROR app - log_exception: Exception on /app/api/v1/metrics [GET]
Traceback (most recent call last):
  File "/opt/acme/python38/lib/python3.8/site-packages/flask/app.py", line 1950, in full_dispatch_request
    rv = self.dispatch_request()
  File "/opt/acme/python38/lib/python3.8/site-packages/flask/app.py", line 1936, in dispatch_request
    return self.view_functions[rule.endpoint](**req.view_args)
  File "/opt/acme/python38/lib/python3.8/site-packages/flask_restx/api.py", line 375, in wrapper
    resp = resource(*args, **kwargs)
  File "/opt/acme/python38/lib/python3.8/site-packages/flask/views.py", line 89, in view
    return self.dispatch_request(*args, **kwargs)
  File "/opt/acme/python38/lib/python3.8/site-packages/flask_restx/resource.py", line 44, in dispatch_request
    resp = meth(*args, **kwargs)
  File "/opt/acme/python38/lib/python3.8/site-packages/flask_restx/marshalling.py", line 248, in wrapper
    resp = f(*args, **kwargs)
  File "/dashboardApp/api/src/controllers/metrics_controller.py", line 44, in get
    return MetricsService().get_data(request.args.get("service"))
  File "/dashboardApp/api/src/service/metrics_service.py", line 149, in get_data
    return db.execute_query(self._services[service]())
  File "/dashboardApp/api/src/utils/db_connector.py", line 97, in execute_query
    self.close_db_connection(connection, cursor)
psycopg2.InterfaceError: connection already closed
2021-08-19 15:09:18 INFO db_connector - get_db_connection: DB Client received
2021-08-19 15:09:18 INFO db_connector - get_db_connection: DB Client received
```

Does this log have structure? It seems like it. Let’s take a line and break it down:

`2021-08-19 15:09:18 INFO db_connector - get_db_connection: DB Client received`

1. We have a timestamp of the format `YYYY-MM-DD HH:MM:SS`
2. Then we have a log level indicator of `INFO`
3. Next comes the module name, `db_connector`
4. A divider, `-`, for readability
5. Then the function name, `get_db_connection`
6. Another divider `:` for readability
7. Lastly the log message, `DB Client received`

We could express the structure with a schema like:

`YYYY-MM-DD HH:MM:SS LOGLEVEL MODULE - FUNCTION: MESSAGE`

So, does the log have structure?

*To humans, yes*. Humans are good at picking out patterns, so we see patterns when we look at these logs and can use those patterns to help us scan the logs and understand what’s going on. We could even think of ways to use `grep` to search through the logs to accomplish certain goals, such as finding all the `ERROR` messages or finding all the messages from `2021-08-19`.

*How about machines*? When we’re talking about Structured Logging as a technical term, we are describing the machine’s perspective into the logs. How does a machine see this log? Does it see structure like humans do? The answer is no, at least without additional information. A machine will see this log as raw text output, and won’t know what to do if you ask it to find all the `ERROR` messages or the messages from a certain day without more information.

And even if you told a machine how to parse these logs, say with `grep`, you’d have to come up with a different command for a different set of logs. For instance, here is an excerpt from Java middleware logs:

```
2021-08-19 16:12:57 +0000 [INFO] from com.acme.infra.analytics.foobar.entityservice.MetricEntityServiceImpl in application-akka.actor.default-dispatcher-445 - # Records found 60
2021-08-19 16:12:57 +0000 [INFO] from org.jdbcdslog.StatementLogger in application-akka.actor.default-dispatcher-445 - select t0.threshold_name, t0.metricMetadataMetric, t0.value from infra_analytics_ui.metric_threshold t0;
2021-08-19 16:12:57 +0000 [INFO] from org.jdbcdslog.StatementLogger in application-akka.actor.default-dispatcher-445 - select t0.threshold_name, t0.metricMetadataMetric, t0.value from infra_analytics_ui.metric_threshold t0;
```

This is a different schema, of the form:

`YYYY-MM-DD HH:MM:SS TIMEZONE [LOGLEVEL] from MODULE in FUNCTION - MESSAGE`

It’s pretty similar to the Python logs above but different enough that you’d need more explicit instructions for a machine to be able to parse it.

To summarize, the logs we’ve seen so far are *structured to humans* but *unstructured to machines*.

What we want is to have all our logs in a centralized location and have them structured such that a machine can quickly dig through millions of log lines and find us what we need. For that, we need structured logging.

## Machine Structured Logging

Let’s look at an example of a machine-structured version of the log excerpts above. Here’s the Python one:

```
{
    "timestamp": "1629390660",
    "request_id": "27348ac0-010b-11ec-9a03-0242ac130003",
    "level": "INFO",
    "message": "DB Client received",
    "traceback": "db_connector.get_db_connection"
}
```

And the Java one:

```
{
    "timestamp": "1629390660",
    "request_id": "27348ac0-010b-11ec-9a03-0242ac130003",
    "level": "INFO",
    "message": "# Records found 60",
    "traceback": "com.acme.infra.analytics.foobar.entityservice.MetricEntityServiceImpl.application-akka.actor.default-dispatcher-445"
}
```

Now these aren’t real examples, in the sense that they didn’t come from existing structured logging frameworks for Java or Python. But they do demonstrate some of the best practices of machine-structured logging:

* It’s a text format, not binary, which means that our centralized logging service (like Splunk) can meaningfully search and analyze it
* It uses clear key-value pairs to allow us to extract fields from log events when we search
* It is still human-readable! (This is a big reason why JSON is preferred for structured logs instead of XML.)
* There are unique identifiers (the `request_id` GUID) so that we can associate log events across a distributed system to a single cause, like an API request

## Value Proposition Redux

Let’s imagine that we had all our systems, services, applications, and pipelines sending their logs to a centralized source like Splunk. Let’s imagine that the logs they were sending were not raw text in `.log` files but actually structured `JSON` data with key-value pairs. Furthermore, let’s imagine that *every log event* included additional key-value pairs that we haven’t even thought about yet:

* `"environment": "prod"`
* `"ENV": { ...object of key-value pairs describing environment variables }`
* `"docker-image": "dashboard-app@4m2c34a"`
* `"hostname": "super-host-132"`
* etc...

Wouldn’t that be great on Production Support? You could slice and dice your logs, search for criteria relevant to your investigation, and a centralized logging service like Splunk would be able to crunch through millions of log events in seconds.

## Further Reading

Splunk has some best practices [here](https://dev.splunk.com/enterprise/docs/developapps/addsupport/logging/loggingbestpractices/).

A good way to get up to speed on the concept of structured logging is to read the examples and documentation for one of the more popular structured logging frameworks in your language of choice:

* Java: [Log4j 2](https://logging.apache.org/log4j/2.x/index.html)
* Python: [structlog](https://github.com/hynek/structlog)

