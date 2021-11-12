---
layout: default
title: Observability
nav_order: 7
---

## Metrics

Metrics on flow and cluster status are collected by default. They can be emitted to several locations based on the environment variables set on the engine.

For metrics with tags, a separate timeseries is created for every combination of tags within a metric. For example `trigger.count` will be reported separately for every `stage` within every `flow`.

The following metrics are reported for all backends.

| Name                             | Unit  | Tags                     | Description                                                                          |
|----------------------------------|-------|--------------------------|--------------------------------------------------------------------------------------|
| vm.memory.total                  | byte  | -                        | Amount of memory consumed by the application VM                                      |
| vm.total_run_queue_lengths.total | last  | -                        | Queue length for all VM operations                                                   |
| vm.total_run_queue_lengths.cpu   | last  | -                        | Queue length for CPU operations                                                      |
| vm.total_run_queue_lengths.io    | last  | -                        | Queue length for IO operations                                                       |
| trigger.count                    | count | flow<br/>stage           | Number of triggering events                                                          |
| stage.retry.count                | count | flow<br/>stage           | Number of retries for a stage                                                        |
| stage.count                      | count | flow<br/>stage<br/>error | Number of times the stage has finished executing, either successfully or on an error |
| flow.total                       | last  | -                        | Number of flows partitioned to this node                                             |
| flow.events.total                | last  | flow                     | Number of events actively executing                                                  |

### StatsD

Metrics are emitted over UDP to the configured statsd host by default. In addition to the common metrics, the following metrics are reported:

| Name           | Unit         | Tags            | Description                                                                                                              |
|----------------|--------------|-----------------|--------------------------------------------------------------------------------------------------------------------------|
| stage.duration | timer | flow<br/>stage<br/>error | Millisecond duration of a given stage, including retries. |


StatsD can be configured with these variables:

| Name                   | Default   | Description                                                                    |
|------------------------|-----------|--------------------------------------------------------------------------------|
| JUMPWIRE_STATSD_HOST   | 127.0.0.1 | Hostname or IP of the statsd collector                                         |
| JUMPWIRE_STATSD_PORT   | 8125      | Port of the statsd collector                                                   |
| JUMPWIRE_STATSD_TYPE   | -         | Special formatting for the statsd metrics. The only allowed value is `datadog` |
| JUMPWIRE_STATSD_PREFIX | jumpwire  | A prefix to prepend to all metric names                                        |

### DataDog

DataDog is configured through the StatsD metrics exporter. `JUMPWIRE_STATSD_TYPE` should be set to `datadog` for compatability with DogStatsD in the Datadog Agent.

When using the `datadog` type, the following metrics will have a different format than the base statsd:

| Name           | Unit         | Tags            | Description                                                                                                              |
|----------------|--------------|-----------------|--------------------------------------------------------------------------------------------------------------------------|
| stage.duration | distribution | flow<br/>stage<br/>error | Millisecond duration of a given stage, including retries. <br /> Bucketed values of `10, 100, 500, 1000, 10_000, 60_000` |


### Prometheus

Enabled by default. Prometheus metrics are exposed on port 9568 at `/metrics`. In addition to the common metrics, the following metrics are reported:

| Name           | Unit         | Tags            | Description                                                                                                              |
|----------------|--------------|-----------------|--------------------------------------------------------------------------------------------------------------------------|
| stage.duration | distribution | flow<br/>stage<br/>error | Millisecond duration of a given stage, including retries. <br /> Bucketed values of `10, 100, 500, 1000, 10_000, 60_000` |

### CloudWatch

Access to the configured CloudWatch namespace is granted through IAM credentials.

In addition to the common metrics, the following metrics are reported:

| Name           | Unit         | Tags            | Description                                                                                                              |
|----------------|--------------|-----------------|--------------------------------------------------------------------------------------------------------------------------|
| stage.duration | summary | flow<br/>stage<br/>error | Millisecond duration of a given stage, including retries. |

CloudWatch can be configured with these variables:

| Name                                 | Default  | Description                                      |
|--------------------------------------|----------|--------------------------------------------------|
| JUMPWIRE_CLOUDWATCH_NAMESPACE        | jumpwire | Which CloudWatch namespace to publish metrics to |
| JUMPWIRE_CLOUDWATCH_INTERVAL_SECONDS | 30       | How often to publish metrics, in seconds         |
| JUMPWIRE_CLOUDWATCH_ENABLED          | false    | Boolean to enable/disable CloudWatch metrics     |

## Logs

Logs are always emitted to stdout/stderr, and can optionally be sent to a remote backend. The follow logging related configuration options are supported:

| Name                            | Default                 | Description                                                                           |
| LOG_LEVEL                       | info                    | Log verbosity. One of debug, info, notice, warning, error, critical, alert, emergency |
| JUMPWIRE_LOG_BACKEND            | -                       | Additional backend to send logs. The only valid value is `cloudwatch`                 |
| JUMPWIRE_CLOUDWATCH_GROUP_NAME  | jumpwire                | Group name in CloudWatch                                                              |
| JUMPWIRE_CLOUDWATCH_STREAM_NAME | `jumpwire@{IP address}` | Name for the CloudWatch stream                                                        |
