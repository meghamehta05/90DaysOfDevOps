Step 1: Go to your repo
cd devops-git-practice
Step 2: Create folder structure
mkdir -p 2026/day-36
Step 3: Move your project files

 (adjust filenames based on your app)

mv Dockerfile 2026/day-36/
mv docker-compose.yml 2026/day-36/
mv .dockerignore 2026/day-36/
mv .env 2026/day-36/
mv README.md 2026/day-36/
mv app.py 2026/day-36/
mv app.js 2026/day-36/
mv requirements.txt 2026/day-36/

 If unsure what exists:

ls
Step 4: Check git status
git status
 Step 5: Stage everything
git add 2026/day-36
Step 6: Commit changes
git commit -m "Day 36 - Dockerized full application with Docker Compose completed"
 Step 7: Push to GitHub
git push origin main
 If you're using another branch
git push origin <branch-name>