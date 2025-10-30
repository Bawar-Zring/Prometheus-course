# What is OpenTelemetry (OTel)?

OpenTelemetry is an open-source observability framework (APIs, SDKs, and tooling) for generating, collecting, processing, and exporting traces, metrics, and logs in a vendor-neutral way. It provides standard APIs and SDKs so applications and libraries can emit telemetry, and a Collector to receive, process, and forward that telemetry to many backends.

Key pieces:
- API & SDKs: used by applications and libraries to instrument and emit telemetry.
- OTLP (OpenTelemetry Protocol): the default binary protocol used to send telemetry between SDKs, agents, and collectors.
- Collector: a standalone service that receives telemetry, processes it, and exports it to backends.

# The Collector pipeline (receivers → processors → exporters)

- Receivers: accept telemetry into the Collector.
  - Examples: otlp (for SDKs), prometheus (to scrape targets), filelog (to read log files).
- Processors: transform or protect the pipeline.
  - Examples: batch, sampling, attributes (add/modify resource or span attributes), resource detection, relabeling.
- Exporters: send telemetry out.
  - Examples: otlp (forward), prometheus_remote_write, loki, logging (debug), jaeger/tempo exporters.

This receiver → processor → exporter pipeline is the canonical model for OpenTelemetry Collector configuration.

# OTel vs Prometheus (short)

- Prometheus uses a pull model (scrape targets) and is a full monitoring system for metrics.
- OpenTelemetry is a broader observability framework (traces, metrics, logs) that typically uses a push model (OTLP).
- The Collector merges both worlds: it can scrape Prometheus endpoints (prometheusreceiver) and convert/forward metrics via OTLP or remote_write.


# Grafana Alloy

Grafana Alloy is Grafana’s high-performance, vendor-neutral distribution of the OpenTelemetry Collector — a curated/packaged collector with added Prometheus-friendly components and production-ready defaults. Treat it as a drop-in collector with extra conveniences for Prometheus ingestion and common Grafana stack integrations.

Benefits (high level):
- Pre-built, tested Collector distribution with commonly used receivers/processors/exporters.
- Easier to deploy with sensible defaults for Prometheus + OpenTelemetry pipelines.
- Designed to integrate well with Grafana backends (Tempo, Loki, Prometheus remote_write, etc.).

