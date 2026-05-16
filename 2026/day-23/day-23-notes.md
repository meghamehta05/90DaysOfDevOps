Step 1: Go to your repo
cd devops-git-practice
 Step 2: Branching commands
# list branches
git branch

# create branch
git branch feature-1

# switch branch
git switch feature-1

# create + switch in one command
git switch -c feature-2

# go back to main
git switch main
Step 3: Make commit on feature-1
git switch feature-1

# edit file (manually), then:
git add .
git commit -m "Added feature-1 changes"
 Step 4: Check main branch
git switch main
git log --oneline
 Step 5: Delete branch (if needed)
git branch -d feature-2
 Step 6: Push to GitHub (FIRST TIME CONNECT)
Add remote repo:
git remote add origin https://github.com/<meghamehta05>/devops-git-practice.git
Push main branch:
git push -u origin main
Push feature branch:
git push -u origin feature-1
Step 7: Pull changes from GitHub
git pull origin main
 Step 8: Fetch vs Pull (command demo)
git fetch origin
git pull origin main
Step 9: Update git-commands.md
git add git-commands.md
git commit -m "Added Git branching and GitHub commands"
git push origin main
 If push fails
git pull origin main --rebase
git push origin main