Step 1: Go to your repo
cd devops-git-practice
Step 2: Create folder structure
mkdir -p 2026/day-35
 Step 3: Move your files into Day 35 folder
mv Dockerfile 2026/day-35/
mv docker-compose.yml 2026/day-35/
mv app.js 2026/day-35/
mv app.go 2026/day-35/
mv day-35-multistage-hub.md 2026/day-35/

 If you are not sure what files exist:

ls
 Step 4: Check repo status
git status
 Step 5: Stage changes
git add 2026/day-35
Step 6: Commit changes
git commit -m "Day 35 - Multi-stage Docker builds and Docker Hub push completed"
 Step 7: Push to GitHub
git push origin main
 If you're working on a different branch
git push origin <branch-name>
 Step 8 (Important) – Docker Hub Push Commands
Login to Docker Hub
docker login
Tag your image
docker tag local-image-name yourusername/image-name:latest
Push image
docker push yourusername/image-name:latest
Verify pull
docker pull yourusername/image-name:latest
Bonus safety check
ls 2026/day-35