Task 1: What is Docker (write in notes)

Add in your markdown:

Container = lightweight isolated environment to run apps
Why needed = solves “it works on my machine” problem
VM vs Container:
VM = full OS (heavy)
Container = shares OS kernel (lightweight)
Docker architecture:
Docker Client (you run commands)
Docker Daemon (background engine)
Images (blueprint)
Containers (running instance)
Registry (Docker Hub)
Task 2: Install & Verify Docker
docker --version
docker run hello-world
 Task 3: Run Real Containers
Nginx (web server)
docker run -d -p 8080:80 --name mynginx nginx

Open:

http://localhost:8080
Ubuntu container (interactive)
docker run -it ubuntu bash
List containers
docker ps
docker ps -a
Stop & remove container
docker stop mynginx
docker rm mynginx
 Task 4: Explore Docker
Detached mode
docker run -d nginx
Named container
docker run -d --name webserver nginx
Port mapping
docker run -d -p 9090:80 nginx
Logs
docker logs webserver
Execute inside container
docker exec -it webserver bash
