# dcos-stats

Routing of metrics from DC/OS.

![architecture diagram](architecture.png)

## Repo contents

- **[module](module/)**: C++ code for the mesos-agent module. This module is installed by default on DC/OS EE 1.7+, with additional input/output support added as of EE 1.8+.
  - Metrics from containers: Containers are each given a unique StatsD endpoint, advertised via `STATSD_UDP_HOST`/`STATSD_UDP_PORT` environment variables. The module then tags and forwards upstream any metrics sent to that endpoint. (EE 1.7+)
  - Metrics from the agent itself: Forwards information about each container's resource utilization ([ResourceStatistics](https://github.com/apache/mesos/blob/master/include/mesos/mesos.proto#L908)), as reported by the agent. (EE 1.8+)
  - Output formats: StatsD to `metrics.marathon.mesos` with tags added via key prefixes or datadog tags (EE 1.7+), and/or Avro metrics sent to a local Collector on TCP port `64113` (EE 1.8+)
- **[collector](collector/)**: A Marathon process which runs on every agent node. Listens on TCP port `64113` for Avro-formatted metrics from the mesos-agent module as well as other processes on the system. Data is collated and forwarded to a Kafka instance, and/or exposed to local partner agents (TBD).
- **[consumer](consumer/)**: Kafka Consumer implementations which fetch Avro-formatted metrics and do something with them (print to `stdout`, write to a database, etc).
- **examples**: Reference implementations of programs which integrate with the metrics stack:
  - **[collector-emitter](examples/collector-emitter/)**: A reference for DC/OS system processes which emit metrics. Sends some Avro metrics data to a local Collector process.
  - **[local-stack](examples/local-stack/)**: Helper scripts for running a full metrics stack on a dev machine. Feeds stats into itself and prints them at the end. Requires a running copy of Zookeeper (reqd by Kafka).
  - **[statsd-emitter](examples/statsd-emitter/)**: A reference for mesos tasks which emit metrics. Sends some StatsD metrics to the `STATSD_UDP_HOST`/`STATSD_UDP_PORT` endpoint advertised by the mesos-agent module.
- **[schema](schema/)**: Avro schemas shared by most everybody that processes metrics (agent module, collector, collector clients, kafka consumers). The exception is containerized processes which only need know how to emit StatsD data.

## Docs

- **[Launching demo processes](DEMO.md)**
- **[Launching the Collector](collector/README.md)**
- [Installing custom module builds (for module dev)](module/README.md)
- [Design doc: Agent module](https://docs.google.com/document/d/11XZF8600Fqfw_yY9YeSh-rX2jJVN4rjw_oQuJFkvlwM/edit#)
- [Design doc: Collector and forwarding via Kafka](https://docs.google.com/document/d/1aJifYTMrmuHnh_zpt8eLbsaU1WP_Fw3M8OvqRf0B6nE/edit#)
