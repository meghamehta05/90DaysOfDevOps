Step 1: Go to repo
cd devops-git-practice
Task 1: Git Merge
Create feature branch
git switch -c feature-login
Add changes + commit
git add .
git commit -m "Login feature work"
git commit -m "Login UI update"
Merge into main
git switch main
git merge feature-login
Create second feature branch
git switch -c feature-signup
git add .
git commit -m "Signup feature work"
Add commit in main BEFORE merge
git switch main
git add .
git commit -m "Main updated before merge"
Merge
git merge feature-signup
Task 2: Git Rebase
Create branch
git switch -c feature-dashboard
git add .
git commit -m "Dashboard step 1"
git commit -m "Dashboard step 2"
Move main ahead
git switch main
git add .
git commit -m "Main update before rebase"
Rebase
git switch feature-dashboard
git rebase main
View graph
git log --oneline --graph --all
 Task 3: Squash vs Merge
Squash merge
git switch main
git merge --squash feature-profile
git commit -m "Squashed feature-profile changes"
Normal merge
git merge feature-settings
 Task 4: Git Stash
Save work
git stash push -m "WIP changes"
List stashes
git stash list
Apply stash
git stash pop
Apply specific stash
git stash apply stash@{0}
Task 5: Cherry Pick
Find commit hash
git log --oneline
Cherry-pick commit
git switch main
git cherry-pick <commit-hash>
 Final Push Commands (Day 24)
git add .
git commit -m "Completed Day 24 advanced Git (merge, rebase, stash, cherry-pick)"
git push origin main