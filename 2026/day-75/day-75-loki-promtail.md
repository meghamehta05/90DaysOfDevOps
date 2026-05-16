TASK 1: Observability Concept (Write in notes)
🔹 What is Observability?

Observability means understanding what is happening inside your system using outputs (metrics, logs, traces) without directly modifying it.

Monitoring → tells WHAT is broken
Observability → tells WHY it is broken
🔹 3 Pillars

1. Metrics (Prometheus)

Numbers over time
CPU, RAM, requests/sec

2. Logs (Loki)

Event-based text records
“Error occurred at line 45”

3. Traces (future tool: Jaeger/OTEL)

Full request journey across services
🔹 Why all 3 are needed
Metrics = detect spike
Logs = find error
Traces = locate service failure point
🔹 Architecture (write in markdown)
Docker Containers
      ↓
  Promtail
      ↓
    Loki
      ↓
   Grafana
      ↓
 User (you debug here)
TASK 2: Loki Setup
Create folder
mkdir loki
loki/loki-config.yml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory
  replication_factor: 1
  path_prefix: /loki

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

storage_config:
  filesystem:
    directory: /loki/chunks
Add to docker-compose.yml
loki:
  image: grafana/loki:latest
  container_name: loki
  ports:
    - "3100:3100"
  volumes:
    - ./loki/loki-config.yml:/etc/loki/loki-config.yml
    - loki_data:/loki
  command: -config.file=/etc/loki/loki-config.yml
  restart: unless-stopped
Add volume
loki_data:
Run Loki
docker compose up -d loki
Test
curl http://localhost:3100/ready

✔ Output:

ready
 TASK 3: Promtail Setup
Create folder
mkdir promtail
promtail/promtail-config.yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker
    static_configs:
      - targets:
          - localhost
        labels:
          job: docker
          __path__: /var/lib/docker/containers/*/*-json.log
    pipeline_stages:
      - docker: {}
Add Promtail in docker-compose.yml
promtail:
  image: grafana/promtail:latest
  container_name: promtail
  volumes:
    - ./promtail/promtail-config.yml:/etc/promtail/promtail-config.yml
    - /var/lib/docker/containers:/var/lib/docker/containers:ro
    - /var/run/docker.sock:/var/run/docker.sock
  command: -config.file=/etc/promtail/promtail-config.yml
  restart: unless-stopped
Restart stack
docker compose up -d
Generate logs
curl http://localhost:8000
 TASK 4: Add Loki to Grafana

Update datasource:

datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    isDefault: true

  - name: Loki
    type: loki
    url: http://loki:3100
Restart Grafana
docker compose restart grafana
 TASK 5: LogQL Queries (VERY IMPORTANT)
1. All logs
{job="docker"}
2. Error logs
{job="docker"} |= "error"
3. Case-insensitive error
{job="docker"} |~ "(?i)error"
4. Notes app logs
{container_name="notes-app"}
5. Count logs
count_over_time({job="docker"}[5m])
6. Top noisy containers
topk(5, sum by (container_name) (rate({job="docker"}[5m])))
 REQUIRED EXERCISE ANSWER

Error logs from notes-app (last 1 hour):

{container_name="notes-app"} |= "error"

Error count per minute:

count_over_time({container_name="notes-app"} |= "error"[1m])
 TASK 6: Grafana Correlation
What you learn here:
Metrics show spike
Logs show reason

Example flow:

CPU spike in Prometheus
Click time range
See logs in Loki
Find error cause instantly
 TASK 7: Important Notes
 Why Loki is different from ELK
ELK → indexes FULL logs (heavy)
Loki → indexes ONLY labels (lightweight)

