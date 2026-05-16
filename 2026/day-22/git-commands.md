Step 1: Create project & files
mkdir devops-git-practice
cd devops-git-practice

git init

touch git-commands.md
touch day-22-notes.md
code .
 Step 2: Configure Git (first time only)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

git config --list
Step 3: First commit
git add git-commands.md day-22-notes.md
git commit -m "Day 22: Initial Git setup and notes file created"
 Step 4: View history
git log --oneline
Step 5: After making changes (repeat 3 times)

Each time you edit git-commands.md, run:

git add git-commands.md
git commit -m "Updated Git commands reference - Day 22 progress"
Step 6: Check changes
git status
git diff
Step 7: Compact history view
git log --oneline --graph --all