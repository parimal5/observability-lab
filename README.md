# Observibility Stack - OSS + Hybrid

## Stack used in Production:

- `Metric`:
  - `Prometheus` - Scrape the metrics and Store in TSDB
  - `Thanos` - Runs as SideCar along side Promehteus and store the metrics for longer period
- `Logs`:
  - `FluentBit` - Log Collector
  - `Loki` - Use to store logs for longer period of time store it as object in S3
- `Tracing`:
  - `OpenTelemetry/OTel Collectors` - The application code incldue OpenTelemetry SDK and OTel Collectors collect the traces
  - `Tempo` - Used to store the traces for longer period of time store as Object in S3
- `Visualization`
  - `Grafana` - Single pane of glass for metrics, logs and traces
    - DataSources: Thanos, Loki and Tempo

## Environment Setup

### Metrics

- We use the helm chart to install the Prometheus Stack which include both `Prometheus` and `Grafana`
- But you can use the seperte Helm charts too since in our setup the Grafana query metrics from Thanos

### Logs

- We use FluentBit to sent logs to Loki.
- Collects:
  - Container stdout/stderr
  - Node logs
  - App logs
- Loki store the logs by labels
- Stores chunks + index in object storage (S3)

### Traces (Application → OpenTelemetry → Tempo)

- Opentelemenety we add to code which will be collected by otel collector and sent to tempo
- Flow:
  - **OpenTelemetry**
    - Instrumentation added to application code (auto or manual)
    - Generates traces (and optionally metrics/logs)
  - **OpenTelemetry Collector**
    - Receives telemetry
    - Batches, processes, exports
  - **Tempo**
    - Stores traces cheaply (object storage)
    - No indexing → traceID based lookup
    - Relies on logs & metrics for correlation

### Visualization - Grafana

- Grafana is special because it is a pure, vendor-neutral query-and-visualization layer that does not own data, does not force an agent, and does not lock you in.
- Grafana:

  - Stores zero metrics, logs, or traces
  - Only speaks query languages:
  - PromQL
  - LogQL
  - TraceQL
  - SQL
  - Cloud APIs

- **Correlation Without Coupling (Quiet Superpower)**

```bash
Logs ↔ Traces
Traces ↔ Metrics
Metrics ↔ Logs
```

> Without backends talking to each other.

How?

- Shared labels
- trace_id
- Time alignment

This works even if:

- Logs in Loki
- races in Tempo
- Metrics in Thanos
- Cloud metrics in CloudWatch

> No other OSS UI does this cleanly.
