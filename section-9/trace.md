## 1. What is a bottleneck (tracing context)

A bottleneck is the operation or service that consumes the largest portion of end-to-end request time. Common bottlenecks:

- Slow DB queries
- External API latency
- Blocking synchronous calls between services
- High CPU/garbage collection pauses in a service

In a trace, the bottleneck usually appears as the span with the longest duration or as a gap in the timeline.

---

## 2. How Grafana Tempo helps

Grafana Tempo collects and stores traces. Typical ingestion sources:

- OpenTelemetry SDKs
- Jaeger clients
- Zipkin clients

Tempo provides a UI (via Grafana) that shows a timeline (Gantt-style) of spans in a single trace. Use it to:

- See which span is longest
- Inspect span attributes/tags and logs
- Correlate traces with Prometheus metrics and Loki logs

---

## 3. Key tracing concepts

- Trace: a single request’s journey across services. Identified by a Trace ID.
- Span: one unit of work in a trace, with start time, duration, name, attributes, and optional child spans.
- Trace context: metadata passed between services (trace_id, span_id, parent_span_id, sampling flags). Example HTTP header used by W3C Trace Context:
  ```
  traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
  ```
- Sampling:
  - Head-based: decide before collecting (e.g., 1% random).
  - Tail-based: decide after seeing full trace (e.g., keep traces > 2s or with errors).
  - Use a mix: light head-based (low overhead) and selective tail-based for large systems.

---

## 4. Correlation: traces, metrics, logs

- Use the trace ID as a field in logs to correlate a request’s log lines with its trace.
- In Grafana:
  - Tempo for traces
  - Prometheus for metrics (CPU, latency, request rates)
  - Loki for logs
- Example: if trace ID `4bf92f...` shows user-service slow, search logs for that trace ID in Loki and metrics around the same timestamp in Prometheus.

