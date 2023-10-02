---
description: >-
  A plugin based on Process Exporter to collect process level of metrics of system
  metrics
---

# Process Exporter Metrics

[Prometheus Node Exporter](https://github.com/prometheus/node_exporter) is a popular way to collect system level metrics from operating systems, such as CPU / Disk / Network / Process statistics.
Fluent Bit 2.2 onwards includes a process exporter plugin that builds off the Prometheus design to collect process level metrics without having to manage two separate processes or agents.

The Process Exporter Metrics plugin implements collecting of the various metrics available from [the 3rd party implementation of Prometheus Process Exporter](https://github.com/ncabatoff/process-exporter) and these will be expanded over time as needed.

**Important note:** Metrics collected with Process Exporter Metrics flow through a separate pipeline from logs and current filters do not operate on top of metrics.

This plugin is only supported on Linux based operating systems.


## Configuration

| Key                       | Description                                                                            | Default   |
| ------------------------- | -------------------------------------------------------------------------------------- | --------- |
| scrape_interval           | The rate at which metrics are collected.                 | 5 seconds |
| path.procfs               | The mount point used to collect process information and metrics                        | /proc/    |
| process\_include\_pattern | regex to determine which processes are included in the metrics produced by this plugin | It is applied for all process unless explicitly set. Default is `.+`. |
| process\_exclude\_pattern | regex to determine which units are excluded in the metrics produced by this plugin     | It is not applied unless explicitly set. Default is `NULL`. |

## Getting Started

### Simple Configuration File

In the following configuration file, the input plugin _process\_exporter\_metrics collects _metrics every 2 seconds and exposes them through our [Prometheus Exporter](../outputs/prometheus-exporter.md) output plugin on HTTP/TCP port 2021.

```
# Process Exporter Metrics + Prometheus Exporter
# -------------------------------------------
# The following example collect host metrics on Linux and expose
# them through a Prometheus HTTP end-point.
#
# After starting the service try it with:
#
# $ curl http://127.0.0.1:2021/metrics
#
[SERVICE]
    flush           1
    log_level       info

[INPUT]
    name            process_exporter_metrics
    tag             process_metrics
    scrape_interval 2

[OUTPUT]
    name            prometheus_exporter
    match           process_metrics
    host            0.0.0.0
    port            2021
```

You can test the expose of the metrics by using _curl:_

```bash
curl http://127.0.0.1:2021/metrics
```

### Container to Collect Host Metrics

When deploying Fluent Bit in a container you will need to specify additional settings to ensure that Fluent Bit has access to the host operating system. The following docker command deploys Fluent Bit with specific mount path for ptocfs and settings enabled to ensure that Fluent Bit can collect from the host. These are then exposed over port 2021.

```
docker run -ti -v /proc:/host/proc \
               -v /sys:/host/sys   \
               -p 2021:2021        \
               fluent/fluent-bit:2.2.0 \
               /fluent-bit/bin/fluent-bit \
                         -i process_exporter_metrics -p path.procfs=/host/proc  \
                         -o prometheus_exporter -p "add_label=host $HOSTNAME" \
                         -f 1
```

## Enhancement Requests

Our current plugin refers a sub-set of the available collectors in the 3rd party implementation of Prometheus Process Exporter, if you would like that we prioritize a specific metrics please open a Github issue by using the following template:\
\
\- [in_process_exporter_metrics](https://github.com/fluent/fluent-bit/issues/new?assignees=\&labels=\&template=feature_request.md\&title=in_process_exporter_metrics:%20add%20ABC%20collector)