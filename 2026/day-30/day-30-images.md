Task 1: Docker Images
Pull images
docker pull nginx
docker pull ubuntu
docker pull alpine
List images
docker images
Compare Ubuntu vs Alpine (write in notes)
Ubuntu = full OS → larger size
Alpine = minimal Linux → very small, lightweight
Used in production for faster builds
Inspect image
docker inspect nginx
Remove image
docker rmi alpine
Task 2: Image Layers
docker image history nginx
Write in notes:
Each instruction in Dockerfile becomes a layer
Layers are cached → faster builds
Shared layers reduce storage usage
Task 3: Container Lifecycle
Create (without starting)
docker create --name mycontainer nginx
Start
docker start mycontainer
Pause / Unpause
docker pause mycontainer
docker unpause mycontainer
Stop
docker stop mycontainer
Restart
docker restart mycontainer
Kill
docker kill mycontainer
Remove
docker rm mycontainer
 Task 4: Working with Containers
Run nginx
docker run -d --name web nginx
Logs
docker logs web
docker logs -f web
Exec into container
docker exec -it web bash
Run single command
docker exec web ls /usr/share/nginx/html
Inspect container
docker inspect web
Task 5: Cleanup
Stop all containers
docker stop $(docker ps -q)
Remove all containers
docker rm $(docker ps -a -q)
Remove unused images
docker image prune -a
Check disk usage
docker system df