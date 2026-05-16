Day 74 Execution Guide (Node Exporter + cAdvisor + Grafana)
1. Final docker-compose.yml (Complete Stack)

Make sure your compose file has ALL services together:

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - "--path.procfs=/host/proc"
      - "--path.sysfs=/host/sys"
      - "--path.rootfs=/rootfs"
    restart: unless-stopped

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    restart: unless-stopped

  grafana:
    image: grafana/grafana-enterprise:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    restart: unless-stopped

  notes-app:
    image: trainwithshubham/notes-app:latest
    container_name: notes-app
    ports:
      - "8000:8000"
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
 2. Final prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]

  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]

  - job_name: "notes-app"
    static_configs:
      - targets: ["notes-app:8000"]
 3. Start Everything
docker compose up -d

Verify:

docker compose ps
 4. Prometheus Checks

Open:

http://localhost:9090/targets

You MUST see:

prometheus → UP
node-exporter → UP
cadvisor → UP
notes-app → UP
 5. Quick PromQL Cheats (Use for screenshots)
CPU Usage (Host)
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Memory Usage %
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
Disk Usage %
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100
Top Container CPU
topk(5, rate(container_cpu_usage_seconds_total{name!=""}[5m]))
 6. Grafana Setup

Open:

http://localhost:3000

Login:

admin / admin123
Add datasource:
URL: http://prometheus:9090
Save & Test → ✔ Success
 7. Dashboard Panels (Minimum Required)
Panel 1: CPU %
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Panel 2: Memory %
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
Panel 3: Disk %
(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100
Panel 4: Container CPU
rate(container_cpu_usage_seconds_total{name!=""}[5m])
Panel 5: Container Memory
container_memory_usage_bytes{name!=""} / 1024 / 1024
Core Concept 
Node Exporter vs cAdvisor
Tool	Focus	Example
Node Exporter	Host machine	CPU, RAM, Disk, Network
cAdvisor	Containers	Docker CPU, memory per container


Node Exporter = OS level
cAdvisor = Container level
Infrastructure
   ↓
Node Exporter (host metrics)
   ↓
cAdvisor (container metrics)
   ↓
Prometheus (data collection)
   ↓
Grafana (visualization)