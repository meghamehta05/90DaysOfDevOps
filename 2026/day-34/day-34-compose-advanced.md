Step 1: Go to repo
cd devops-git-practice
 Step 2: Create folder structure
mkdir -p 2026/day-34
 Step 3: Move all files into Day 34 folder
mv docker-compose.yml 2026/day-34/
mv Dockerfile 2026/day-34/
mv app.py 2026/day-34/
mv day-34-compose-advanced.md 2026/day-34/

If your files have different names, check first:

ls
 Step 4: Check changes
git status
 Step 5: Stage everything
git add 2026/day-34
 Step 6: Commit
git commit -m "Day 34 - Advanced Docker Compose multi-service stack completed"
 Step 7: Push to GitHub
git push 