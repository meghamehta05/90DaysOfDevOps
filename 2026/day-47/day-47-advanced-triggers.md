.github/workflows/pr-lifecycle.yml
name: PR Lifecycle Events

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  pr-info:
    runs-on: ubuntu-latest

    steps:
      - name: Print event type
        run: echo "Event: ${{ github.event.action }}"

      - name: PR details
        run: |
          echo "Title: ${{ github.event.pull_request.title }}"
          echo "Author: ${{ github.event.pull_request.user.login }}"
          echo "From: ${{ github.head_ref }}"
          echo "To: ${{ github.base_ref }}"

      - name: Run only if merged
        if: github.event.action == 'closed' && github.event.pull_request.merged == true
        run: echo "PR was merged successfully 🎉"
 .github/workflows/pr-checks.yml
name: PR Validation Checks

on:
  pull_request:
    branches: [main]

jobs:
  branch-name-check:
    runs-on: ubuntu-latest

    steps:
      - name: Check branch name
        run: |
          BRANCH="${{ github.head_ref }}"
          echo "Branch: $BRANCH"

          if [[ ! "$BRANCH" =~ ^(feature|fix|docs)/ ]]; then
            echo "Invalid branch name"
            exit 1
          fi

  pr-body-check:
    runs-on: ubuntu-latest

    steps:
      - name: Check PR body
        run: |
          BODY="${{ github.event.pull_request.body }}"
          if [ -z "$BODY" ]; then
            echo "Warning: PR body is empty"
          fi

  file-size-check:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Check file sizes
        run: |
          find . -type f -size +1M && echo "File too large!" && exit 1 || echo "All files OK"
 .github/workflows/scheduled-tasks.yml
name: Scheduled Tasks

on:
  schedule:
    - cron: "30 2 * * 1"
    - cron: "0 */6 * * *"
  workflow_dispatch:

jobs:
  cron-job:
    runs-on: ubuntu-latest

    steps:
      - name: Show trigger
        run: echo "Triggered by schedule: ${{ github.event.schedule }}"

      - name: Health check
        run: |
          curl -I https://example.com || echo "Site unreachable"
 .github/workflows/smart-triggers.yml
name: Smart Triggers

on:
  push:
    branches:
      - main
      - "release/*"
    paths:
      - "src/**"
      - "app/**"

jobs:
  smart-job:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Run smart pipeline
        run: echo "Triggered by relevant code change"
 .github/workflows/smart-triggers-ignore.yml
name: Smart Ignore Workflow

on:
  push:
    branches:
      - main
      - "release/*"
    paths-ignore:
      - "*.md"
      - "docs/**"

jobs:
  ignore-job:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Docs change ignored workflow triggered"
 .github/workflows/tests.yml
name: Run Tests

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Run tests
        run: echo "Running tests..."
 .github/workflows/deploy-after-tests.yml
name: Deploy After Tests

on:
  workflow_run:
    workflows: ["Run Tests"]
    types: [completed]

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest

    steps:
      - name: Deploy step
        run: echo "Deploying after successful tests "

  skip:
    if: ${{ github.event.workflow_run.conclusion != 'success' }}
    runs-on: ubuntu-latest

    steps:
      - name: Skip deploy
        run: echo "Tests failed, skipping deployment "
 .github/workflows/external-trigger.yml
name: External Trigger Workflow

on:
  repository_dispatch:
    types: [deploy-request]

jobs:
  external:
    runs-on: ubuntu-latest

    steps:
      - name: Print payload
        run: echo "Environment: ${{ github.event.client_payload.environment }}"
📄 day-47-advanced-triggers.md
# Day 47 – Advanced Triggers in GitHub Actions

---

## 1. PR Lifecycle Events

### Learned:
- `opened`, `synchronize`, `reopened`, `closed`
- Access PR metadata via `github.event.pull_request`

### Key Insight:
- Helps track full PR lifecycle

---

## 2. PR Validation

### What I built:
- Branch name validator
- PR body checker
- File size checker

### Learning:
- CI can enforce coding standards before merge

---

## 3. Cron Schedules

### Cron used:
- Every Monday 2:30 AM → `30 2 * * 1`
- Every 6 hours → `0 */6 * * *`

### Notes:
- Weekday 9 AM IST → `30 3 * * 1-5`
- First day of month midnight → `0 0 1 * *`

---

## 4. Path Filters

### Learned:
- `paths` → only run on specific file changes
- `paths-ignore` → skip unnecessary changes

---

## 5. Workflow Chain

### workflow_run:
- Runs after another workflow completes
- Depends on success/failure result

### Difference:
- workflow_call → reusable workflow
- workflow_run → event-based chaining

---

## 6. External Trigger

### repository_dispatch:
- Trigger workflows from external systems
- Useful for:
  - Slack bots
  - Monitoring alerts
  - CI/CD automation tools

---

## Final Takeaway

GitHub Actions is not just CI/CD — it is an **event-driven automation engine**.


 Learned 


PR gates (production quality checks)
Event-driven pipelines
Scheduled automation
Cross-workflow chaining
External system triggers