1. OpenTelemetry Architecture

OpenTelemetry is a vendor-neutral observability framework used to collect:

Metrics
Logs
Traces

It does NOT store data. It only collects and exports it.

 OTEL Collector Pipeline
Receivers → Processors → Exporters
✔ Receivers

Accept telemetry data:

OTLP (gRPC / HTTP)
Prometheus format
Jaeger traces
✔ Processors

Modify or optimize data:

batching
filtering
sampling
✔ Exporters

Send data to backend systems:

Prometheus (metrics)
Loki / logging tools
Debug console
Jaeger / Tempo (traces)
 2. OTEL Collector Config
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:

exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
  debug:
    verbosity: detailed

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]

    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]

    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
 3. OTLP Concept

OTLP (OpenTelemetry Protocol) is:

Standard format for telemetry transfer
Supports:
gRPC (4317)
HTTP (4318)

It unifies all observability data into one format.

 4. Distributed Tracing

A trace shows the full journey of a request

Example:

User Request
   ↓
API Gateway (Span 1)
   ↓
Auth Service (Span 2)
   ↓
Database (Span 3)

Each span contains:

trace ID
span ID
timing
metadata

 Helps identify where delay or failure happens

 5. Prometheus Alerting Rules
 Alert File: alert-rules.yml
groups:
  - name: system-alerts
    rules:

      - alert: HighCPUUsage
        expr: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage > 80%"

      - alert: HighMemoryUsage
        expr: (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 > 85
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage detected"

      - alert: ContainerDown
        expr: absent(container_last_seen{name="notes-app"})
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container is down"

      - alert: TargetDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Scrape target is down"

      - alert: HighDiskUsage
        expr: (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk usage high"
 6. Alert Types Explained
Alert	Meaning
CPU High	System overloaded
Memory High	RAM pressure
ContainerDown	App stopped
TargetDown	Prometheus cannot scrape
Disk High	Storage risk
 7. Prometheus vs Grafana Alerts
 Prometheus Alerts
Rule-based evaluation engine
Detects conditions
Needs Alertmanager (or Grafana) for notifications
 Grafana Alerts
UI-based alert system
Easier setup
Supports email, Slack, Teams, etc.

 Prometheus = detection
 Grafana = notification layer

 8. OTEL Collector Test Trace

Trace flow:

curl → OTLP endpoint → Collector → Debug exporter

Logs show:

trace ID
span name
timing
attributes
 9. Metrics Flow via OTEL
Application → OTEL Collector → Prometheus → Grafana

Example metric:

test_requests_total = 42
 10. Full Observability Architecture
METRICS:
Node Exporter → Prometheus → Grafana

CONTAINERS:
cAdvisor → Prometheus → Grafana

TRACES:
App → OTEL Collector → Debug / Future Jaeger

LOGS:
Docker → Promtail → Loki → Grafana

ALERTING:
Prometheus + Grafana → Notifications
 11. Key Learnings
OTEL unifies telemetry formats
Collector acts as middleware pipeline
Traces help debug request flow
Alerts prevent manual monitoring
Grafana is the central observability hub

 12. Final Summary

Today I built:

OpenTelemetry Collector
OTLP trace ingestion
Metrics pipeline via OTEL → Prometheus
Prometheus alerting rules
Grafana alerting setup
Full observability integration