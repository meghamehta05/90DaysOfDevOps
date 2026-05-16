day-39-cicd-concepts.md
# Day 39 – CI/CD Concepts

## 1. Problem in Manual Deployment

### What can go wrong?
- Code conflicts when multiple developers push changes
- Human errors during deployment
- Missed steps (build, test, config)
- Downtime due to incorrect releases
- No rollback strategy
- Environment mismatch (dev vs prod)

---

### “It works on my machine”
This means:
- Code runs locally but fails in production

Why it happens:
- Missing dependencies on server
- Different OS/environment versions
- Hardcoded paths or configs
- Untested edge cases

---

### Manual deployment frequency
Safe manual deployment is:
- 1–2 times/day max for small teams
- Anything more increases risk of failure

---

## 2. CI vs CD

### Continuous Integration (CI)
- Developers frequently merge code into a shared repo
- Each merge triggers automated build + tests
- Finds bugs early

Example:
- GitHub push triggers automated test pipeline

---

### Continuous Delivery (CD)
- Code is always ready to deploy
- After CI, deployment to staging is automatic or manual approval

Example:
- After passing tests, code is deployed to staging server

---

### Continuous Deployment
- Every passing change is automatically deployed to production
- No manual approval needed

Example:
- Netflix deploys updates automatically after tests pass

---

## 3. Pipeline Anatomy

### Trigger
Event that starts pipeline (push, PR, schedule)

### Stage
Logical group of jobs (build, test, deploy)

### Job
Set of steps executed on a runner

### Step
Single command/action (npm install, docker build)

### Runner
Machine that executes jobs

### Artifact
Output generated (build file, Docker image)

---

## 4. Pipeline Diagram


Developer Push Code
↓
GitHub Repo
↓ (Trigger)
CI Pipeline
↓
Build Stage
↓
Test Stage
↓
Docker Image Build
↓
Push to Registry
↓
Deploy to Staging Server


---

## 5. Open Source CI/CD Example

Repository used: GitHub Actions workflow example

### Observations:
- Trigger: push and pull_request
- Jobs: usually 2–3 jobs (build, test, lint)
- Purpose:
  - Install dependencies
  - Run tests
  - Check code quality
  - Build project

---

## 6. Key Learnings

- CI/CD automates software delivery
- CI = integration + testing
- CD = delivery/deployment
- Pipelines reduce human error
- Automation = faster + safer releases