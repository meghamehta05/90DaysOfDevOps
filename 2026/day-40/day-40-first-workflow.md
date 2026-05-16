.github/workflows/hello.yml
name: Hello GitHub Actions

on:
  push:

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Hello message
        run: echo "Hello from GitHub Actions!"

      - name: Show date and time
        run: date

      - name: Show branch name
        run: echo "Branch: ${{ github.ref_name }}"

      - name: List files
        run: ls -la

      - name: Show OS
        run: uname -a

      # Uncomment below to test failure
      # - name: Force failure
      #   run: exit 1

 day-40-first-workflow.md
# Day 40 – First GitHub Actions Workflow

## Workflow File

```yaml
name: Hello GitHub Actions

on:
  push:

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Hello message
        run: echo "Hello from GitHub Actions!"

      - name: Show date and time
        run: date

      - name: Show branch name
        run: echo "Branch: ${{ github.ref_name }}"

      - name: List files
        run: ls -la

      - name: Show OS
        run: uname -a
GitHub Actions Anatomy
on:

Defines the trigger event (here: push)

jobs:

Collection of tasks that run in parallel or sequence

runs-on:

Defines the OS environment (ubuntu-latest)

steps:

Individual tasks inside a job

uses:

Uses a prebuilt GitHub Action (like checkout code)

run:

Executes shell commands

name:

Human-readable label for steps

What I Learned
GitHub Actions runs pipelines in the cloud
Every push triggers automation
Steps execute one by one on a runner
${{ github.ref_name }} gives branch name
Logs help debug failures easily
Failure Test Insight
When a command fails, pipeline turns  red
Logs show exact step where error happened
Fixing and re-pushing reruns pipeline
Final Result

My first CI pipeline successfully ran on GitHub Actions 


---

#  Now Push It

Run:

```bash id="push40"
git add .
git commit -m "Day 40 first GitHub Actions workflow"
git push origin main