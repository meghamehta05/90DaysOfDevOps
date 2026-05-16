Step 1: Push your work (Day 26)
git add .
git commit -m "Completed Day 26 GitHub CLI (gh) hands-on practice"
git push origin main
If push fails
git pull origin main --rebase
git push origin main
 Optional (important for GitHub CLI workflow)
Login to GitHub CLI
gh auth login
Check login status
gh auth status
Common useful gh commands (add to git-commands.md)
Repo commands
gh repo create my-repo --public --clone
gh repo list
gh repo view
gh repo view --web
Clone using gh
gh repo clone owner/repo
Issues
gh issue create
gh issue list
gh issue view 1
gh issue close 1
Pull Requests
gh pr create
gh pr list
gh pr view
gh pr merge
Workflows
gh run list
gh workflow list
gh run view