TASK 1 — Clone & Start Stack
git clone https://github.com/LondheShubham153/observability-for-devops.git
cd observability-for-devops
docker compose up -d

Check running containers:

docker compose ps
 TASK 2 — Verify All Services

Open:

Prometheus → http://localhost:9090
Grafana → http://localhost:3000 (admin/admin)
Node Exporter → http://localhost:9100/metrics
cAdvisor → http://localhost:8080
Loki → http://localhost:3100/ready
Notes App → http://localhost:8000

Check logs:

docker logs otel-collector
 TASK 3 — Validate METRICS (Prometheus)

Go to:
 http://localhost:9090/targets

Ensure ALL are UP:

prometheus
node-exporter
cadvisor
otel-collector

Run queries:

up
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
topk(3, container_memory_usage_bytes{name!=""})
 TASK 4 — Validate LOGS (Loki + Promtail)

Generate logs:

for i in $(seq 1 50); do curl -s http://localhost:8000 > /dev/null; done

In Grafana → Explore → Loki:

{job="docker"}
{container_name="notes-app"}
{job="docker"} |= "error"
sum by (container_name) (rate({job="docker"}[5m]))
 TASK 5 — Validate TRACES (OTEL)

Send test trace:

curl -X POST http://localhost:4318/v1/traces ...

Check logs:

docker logs otel-collector | grep -A 20 "GET /api/notes"
 TASK 6 — Build Grafana Dashboard

Go to:
 http://localhost:3000

Create dashboard:

 System Health
CPU:
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Memory:
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
Disk:
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100
 Containers
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100
container_memory_usage_bytes{name!=""} / 1024 / 1024
 Logs Panel
{container_name="notes-app"}
 TASK 7 — Verify FULL FLOW
Test full pipeline:
Hit app:
curl http://localhost:8000
Check:
Metrics → Prometheus 
Logs → Loki 
Traces → OTEL 
Dashboard → Grafana 
 TASK 8 — Final Validation Checklist

Make sure:

 All containers running
 Prometheus targets UP
 Grafana loads dashboards
 Loki returns logs
 OTEL collector shows traces
 Notes app accessible
 FINAL OUTPUT

we have:

 Metrics pipeline

Prometheus → Grafana

 Logs pipeline

Promtail → Loki → Grafana

 Traces pipeline

App → OTEL Collector → Debug output

