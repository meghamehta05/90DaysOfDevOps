Task 1: Data Loss Problem
Run Postgres
docker run -d --name pg1 -e POSTGRES_PASSWORD=123 postgres
Enter container
docker exec -it pg1 psql -U postgres
After stopping/removing
docker stop pg1
docker rm pg1
Observation (write in notes)
Data is lost because container filesystem is temporary (ephemeral)
Task 2: Named Volumes
Create volume
docker volume create pgdata
Run container with volume
docker run -d \
--name pg2 \
-e POSTGRES_PASSWORD=123 \
-v pgdata:/var/lib/postgresql/data \
postgres
Check volumes
docker volume ls
docker volume inspect pgdata
After restart test
docker stop pg2
docker rm pg2
docker run -d \
--name pg3 \
-e POSTGRES_PASSWORD=123 \
-v pgdata:/var/lib/postgresql/data \
postgres
 Task 3: Bind Mount
Create folder
mkdir web
echo "<h1>Docker Bind Mount</h1>" > web/index.html
Run Nginx with bind mount
docker run -d -p 8080:80 \
-v $(pwd)/web:/usr/share/nginx/html \
nginx
 Notes:
Named volume = managed by Docker
Bind mount = directly uses host filesystem
 Task 4: Default Network
docker network ls
docker network inspect bridge
Run containers
docker run -dit --name c1 alpine sh
docker run -dit --name c2 alpine sh
Try ping
docker exec c1 ping c2
 Task 5: Custom Network
Create network
docker network create my-app-net
Run containers
docker run -dit --name app1 --network my-app-net alpine sh
docker run -dit --name app2 --network my-app-net alpine sh
Ping by name
docker exec app1 ping app2
 Task 6: Full Setup
DB + App network
docker run -d --name db \
--network my-app-net \
-v pgdata:/var/lib/postgresql/data \
postgres
docker run -dit --name app --network my-app-net alpine sh
 Cleanup
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
docker volume prune -f