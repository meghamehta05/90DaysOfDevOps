# Docker Cheat Sheet

## Container Commands
- `docker run image` → Run a container
- `docker run -it image` → Interactive container
- `docker run -d image` → Detached mode
- `docker ps` → List running containers
- `docker ps -a` → List all containers
- `docker stop container_id` → Stop container
- `docker start container_id` → Start container
- `docker restart container_id` → Restart container
- `docker rm container_id` → Remove container
- `docker exec -it container bash` → Enter container shell
- `docker logs container_id` → View logs

---

## Image Commands
- `docker build -t name:tag .` → Build image
- `docker images` → List images
- `docker rmi image_id` → Remove image
- `docker pull image` → Pull from Docker Hub
- `docker push image` → Push to Docker Hub
- `docker tag image new_name` → Tag image

---

## Volume Commands
- `docker volume create vol_name` → Create volume
- `docker volume ls` → List volumes
- `docker volume inspect vol_name` → Inspect volume
- `docker volume rm vol_name` → Remove volume

---

## Network Commands
- `docker network create net_name` → Create network
- `docker network ls` → List networks
- `docker network inspect net_name` → Inspect network
- `docker network connect net container` → Connect container

---

## Compose Commands
- `docker compose up` → Start services
- `docker compose up -d` → Detached mode
- `docker compose down` → Stop services
- `docker compose down -v` → Stop + remove volumes
- `docker compose ps` → List services
- `docker compose logs` → View logs
- `docker compose build` → Build services

---

## Cleanup Commands
- `docker system df` → Show disk usage
- `docker system prune` → Remove unused data
- `docker image prune` → Remove unused images
- `docker container prune` → Remove stopped containers

---

## Dockerfile Instructions
- `FROM` → Base image
- `RUN` → Execute commands
- `COPY` → Copy files
- `ADD` → Copy + extract archives
- `WORKDIR` → Set working directory
- `EXPOSE` → Open port
- `CMD` → Default command
- `ENTRYPOINT` → Fixed execution command

---

## Quick Notes
- Image = Blueprint
- Container = Running instance of image
- Volumes = Persistent storage
- Networks = Container communication layer
# Day 37 – Docker Revision

## Self-Assessment Checklist

- Run a container from Docker Hub → can do
- List, stop, remove containers/images → can do
- Image layers & caching → shaky
- Dockerfile basics → can do
- CMD vs ENTRYPOINT → shaky
- Build & tag image → can do
- Named volumes → can do
- Bind mounts → shaky
- Custom networks → can do
- Docker Compose → can do
- Env variables & .env → shaky
- Multi-stage builds → haven't done
- Push image to Docker Hub → can do
- Healthchecks & depends_on → haven't done

---

## Quick-Fire Answers

### 1. Image vs Container
Image is a template; container is a running instance of that image.

### 2. Data after removing container
Data inside container is lost unless stored in volumes or bind mounts.

### 3. Same network communication
Containers communicate using container name as hostname.

### 4. docker compose down -v vs down
`-v` removes volumes too; normal down keeps them.

### 5. Why multi-stage builds?
They reduce image size by separating build and runtime environments.

### 6. COPY vs ADD
COPY = simple file copy
ADD = copy + supports tar extraction + URLs

### 7. -p 8080:80
Maps host port 8080 → container port 80

### 8. Docker disk usage
`docker system df`

---

## Weak Areas to Revisit

### 1. Image Layers & Caching
- Each RUN creates a layer
- Docker caches unchanged layers
- Faster rebuilds when no changes above layer

### 2. CMD vs ENTRYPOINT
- CMD → default arguments (can override)
- ENTRYPOINT → fixed command (hard to override)

---

## Hands-on Rework

### Task 1: Bind Mount Practice
```bash
docker run -v $(pwd):/app node
environment:
  - NODE_ENV=production