.github/workflows/multi-job.yml
name: Multi Job Workflow

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build step
        run: echo "Building the app"

  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Test step
        run: echo "Running tests"

  deploy:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Deploy step
        run: echo "Deploying application"
  .github/workflows/env-vars.yml
name: Environment Variables Demo

on:
  push:

env:
  APP_NAME: myapp

jobs:
  demo:
    runs-on: ubuntu-latest
    env:
      ENVIRONMENT: staging

    steps:
      - name: Print env vars
        env:
          VERSION: 1.0.0
        run: |
          echo "APP_NAME: $APP_NAME"
          echo "ENVIRONMENT: $ENVIRONMENT"
          echo "VERSION: $VERSION"

      - name: GitHub Context Variables
        run: |
          echo "Commit SHA: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
 .github/workflows/job-outputs.yml
name: Job Outputs Demo

on:
  push:

jobs:
  generate:
    runs-on: ubuntu-latest
    outputs:
      date: ${{ steps.set-date.outputs.date }}

    steps:
      - name: Set date output
        id: set-date
        run: echo "date=$(date)" >> $GITHUB_OUTPUT

  consume:
    runs-on: ubuntu-latest
    needs: generate

    steps:
      - name: Print output from previous job
        run: echo "Date is ${{ needs.generate.outputs.date }}"
 .github/workflows/conditionals.yml
name: Conditionals Demo

on:
  push:

jobs:
  conditional-job:
    runs-on: ubuntu-latest

    steps:
      - name: Run only on main branch
        if: github.ref == 'refs/heads/main'
        run: echo "This runs only on main branch"

      - name: Failing step (for demo)
        id: fail-step
        run: exit 1
        continue-on-error: true

      - name: Runs only if previous failed
        if: failure()
        run: echo "Previous step failed"

  pr-only-job:
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - name: Push only job
        run: echo "This runs only on push events"
 .github/workflows/smart-pipeline.yml
name: Smart Pipeline

on:
  push:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Lint step
        run: echo "Linting code..."

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Test step
        run: echo "Running tests..."

  summary:
    runs-on: ubuntu-latest
    needs: [lint, test]

    steps:
      - name: Pipeline summary
        run: |
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit message: ${{ github.event.head_commit.message }}"
 day-43-jobs-steps.md
# Day 43 – Jobs, Steps, Env Vars & Conditionals

---

## 1. Multi-Job Workflow

### What I learned:
- `needs:` controls job order
- Jobs run in dependency chain:
  build → test → deploy

### Why it matters:
- Ensures code is tested before deployment
- Prevents broken releases

---

## 2. Environment Variables

### Levels:
- Workflow level → global
- Job level → specific to job
- Step level → temporary override

### GitHub context variables:
- `${{ github.sha }}` → commit ID
- `${{ github.actor }}` → who triggered workflow

---

## 3. Job Outputs

### What is it?
- Passing data between jobs

### Why used?
- Share computed values (dates, build IDs, artifacts)

---

## 4. Conditionals

### Key concepts:
- `if:` controls execution
- `failure()` runs only if previous step fails
- `continue-on-error: true` ignores failure

### Use cases:
- Run steps only in main branch
- Handle errors gracefully

---

## 5. Smart Pipeline

### What I built:
- Parallel jobs (lint + test)
- Summary job after completion
- Printed branch and commit info

---

## Final Takeaways

- `needs:` controls workflow order
- `env:` defines variables at different levels
- `outputs:` pass data between jobs
- `if:` adds logic to pipelines
- CI pipelines are programmable workflows, not just script