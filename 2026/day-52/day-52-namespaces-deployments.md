Step 1: Check your repo status

Run this inside your project folder:

git status
Step 2: Add your files

If you created Day 50–52 files:

git add .
Step 3: Commit changes
git commit -m "Day 50-52 Kubernetes progress"
 Step 4: Push to GitHub (IMPORTANT)

Use:

git push origin main

 If your branch is master instead:

git push origin master
 If you get error (very common)
1. “fatal: not a git repository”

Fix:

git init
2. “origin not found”

Fix:

git remote add origin https://github.com/<your-username>/<repo-name>.git

Then:

git push -u origin main
3. First time push asks login

Use:

GitHub username
Personal Access Token (not password)
 Quick shortcut

If everything is already set:

git add . && git commit -m "update" && git push