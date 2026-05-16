.github/workflows/pr-check.yml
name: PR Check

on:
  pull_request:
    branches: [main]

jobs:
  pr-check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Print PR branch
        run: echo "PR check running for branch: ${{ github.head_ref }}"

 .github/workflows/manual.yml
name: Manual Trigger Workflow

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Choose environment (staging/production)"
        required: true
        default: staging

jobs:
  manual-job:
    runs-on: ubuntu-latest

    steps:
      - name: Print environment
        run: echo "Selected environment: ${{ inputs.environment }}"
 .github/workflows/matrix.yml
name: Matrix Build

on:
  push:

jobs:
  test-matrix:
    runs-on: ${{ matrix.os }}

    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ["3.10", "3.11", "3.12"]
        exclude:
          - os: windows-latest
            python-version: "3.10"

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Show Python version
        run: python --version

 day-41-triggers.md
# Day 41 – GitHub Actions Triggers & Matrix Builds

---

## 1. PR Trigger Workflow

### File: pr-check.yml
```yaml
on:
  pull_request:
    branches: [main]
What I learned:
Runs only when PR is opened or updated
Useful for code review validation
Shows status on PR page automatically
2. Cron Schedule Trigger
Example:
on:
  schedule:
    - cron: "0 0 * * *"
Answer:
Every Monday at 9 AM UTC:
0 9 * * 1
3. Manual Trigger
File: manual.yml
Trigger: workflow_dispatch
Accepts input: environment
Learning:
Workflows can be run manually
Inputs allow dynamic execution
Useful for production deployments
4. Matrix Builds
File: matrix.yml
Runs multiple environments in parallel:
Python 3.10, 3.11, 3.12
Ubuntu + Windows
What I observed:
Jobs run in parallel
Each combination is a separate job
Faster testing across environments
5. Exclude & Fail-Fast
Learning:
exclude removes specific combinations
fail-fast: false keeps other jobs running even if one fails
Difference:
fail-fast: true → stops all jobs if one fails
fail-fast: false → runs all jobs independently
Final Takeaways
GitHub Actions supports multiple triggers (push, PR, schedule, manual)
Matrix builds = test across environments automatically
CI pipelines are highly configurable and powerful

---

#  Now Push Everything

Run:

```bash id="push41"
git add .
git commit -m "Day 41 triggers and matrix builds completed"
git push origin main