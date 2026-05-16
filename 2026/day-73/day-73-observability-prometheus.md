1. Observability (Core Concept Notes)
 What is Observability?
Observability = ability to understand system behavior from outputs.

Monitoring → tells something is wrong
Observability → tells why it is wrong

 Monitoring is reactive
Observability is investigative

 3 Pillars of Observability
1. Metrics (Numbers over time)

Examples:

CPU usage
Request rate
Error count

Tools:

Prometheus
CloudWatch
2. Logs (Event records)

Examples:

“User login failed”
Stack traces

Tools:

ELK Stack
Loki
3. Traces (Request journey)

Examples:

API → service → DB latency breakdown

Tools:

OpenTelemetry
Jaeger
 Why all 3 matter
Pillar	Answer
Metrics	What broke
Logs	Why it broke
Traces	Where it broke
 2. Prometheus Setup (Docker)
 Folder
mkdir observability-stack && cd observability-stack
 prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]
       docker-compose.yml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prom_data:/prometheus
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    restart: unless-stopped

volumes:
  prom_data:
 Run
docker compose up -d
Verify

Open:

http://localhost:9090

Go to:

Status → Targets → should show UP
 3. Prometheus Concepts
Metric Types
Counter (only increases)

Example:

total requests
total errors
Gauge (up/down)

Example:

CPU usage
memory usage
Histogram
latency distribution
Summary
percentiles (p95, p99)
Labels example
http_requests_total{method="GET", status="200"}
 4. PromQL Basics
Check health
up
Memory usage
process_resident_memory_bytes
Rate of requests
rate(prometheus_http_requests_total[5m])
Total request rate
sum(rate(prometheus_http_requests_total[5m]))
Filter failed requests
rate(prometheus_http_requests_total{code!="200"}[5m])
Memory in MB
process_resident_memory_bytes / 1024 / 1024
Top 5 metrics
topk(5, prometheus_http_requests_total)
 Answer (important exercise)

 Non-200 request rate:

rate(prometheus_http_requests_total{code!="200"}[5m])
 5. Add Sample App
docker-compose update
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prom_data:/prometheus

  notes-app:
    image: trainwithshubham/notes-app:latest
    ports:
      - "8000:8000"

volumes:
  prom_data:
prometheus.yml update
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: "notes-app"
    static_configs:
      - targets: ["notes-app:8000"]
Restart
docker compose up -d
Generate traffic
curl http://localhost:8000
6. Prometheus Storage
Check disk usage
docker exec prometheus du -sh /prometheus
Retention setting
command:
  - --storage.tsdb.retention.time=15d
What happens when full?
Old metrics get deleted
Prometheus uses rolling time-series storage
 7. Key Concepts Summary
Pull model

Prometheus scrapes targets (not push)

up metric
1 → healthy
0 → down
Rule

 Always use:

rate() → then sum()
 8. Architecture (Important for exam/interview)
[App] → Metrics → [Prometheus] → Queries (PromQL)
                    ↓
                 Storage (TSDB)
9. What You Built Today

 Prometheus running
 Metrics scraping
 PromQL queries
 Time-series database
 Observability foundation