Task 1: First Dockerfile
 Create Dockerfile
FROM ubuntu

RUN apt-get update && apt-get install -y curl

CMD ["echo", "Hello from my custom image!"]
 Build image
docker build -t my-ubuntu:v1 .
Run container
docker run my-ubuntu:v1
 Task 2: Full Dockerfile Example
FROM ubuntu

RUN apt-get update && apt-get install -y curl

WORKDIR /app

COPY . /app

EXPOSE 8080

CMD ["echo", "Dockerfile complete"]
Build & run
docker build -t full-image:v1 .
docker run full-image:v1
 Task 3: CMD vs ENTRYPOINT
CMD example
FROM ubuntu
CMD ["echo", "hello"]
Run
docker run cmd-image world
ENTRYPOINT example
FROM ubuntu
ENTRYPOINT ["echo"]
Run
docker run entry-image hello world
 Task 4: Nginx Web Image
Create file
echo "<h1>My Docker Website</h1>" > index.html
Dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
Build & run
docker build -t my-website:v1 .
docker run -d -p 8080:80 my-website:v1
 Task 5: .dockerignore
Create file
touch .dockerignore
Add content:
node_modules
.git
*.md
.env
 Task 6: Build Cache Insight
docker build -t test-cache:v1 .
docker build -t test-cache:v2 .
 Key Notes (write in markdown)
Docker layers = cache units
Order matters → stable steps first
COPY too early breaks caching efficiency