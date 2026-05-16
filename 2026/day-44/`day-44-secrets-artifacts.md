.github/workflows/secrets-demo.yml
name: Secrets Demo

on:
  push:

jobs:
  secrets-check:
    runs-on: ubuntu-latest

    steps:
      - name: Show secret status
        run: echo "The secret is set: ${{ secrets.MY_SECRET_MESSAGE != '' }}"

      - name: Use secret safely
        env:
          SECRET_MSG: ${{ secrets.MY_SECRET_MESSAGE }}
        run: |
          echo "Secret received successfully"
          echo "DO NOT PRINT SECRET CONTENT"
📄 .github/workflows/artifacts-demo.yml
name: Artifacts Demo

on:
  push:

jobs:
  generate:
    runs-on: ubuntu-latest

    steps:
      - name: Create file
        run: echo "Hello from artifact job" > output.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: my-file
          path: output.txt

  consume:
    runs-on: ubuntu-latest
    needs: generate

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: my-file

      - name: Read file
        run: cat output.txt
📄 .github/workflows/real-test-ci.yml
name: Real Test CI

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run script
        run: |
          echo "Running test script..."
          chmod +x test.sh || true
          ./test.sh
📄 .github/workflows/cache-demo.yml
name: Cache Demo

on:
  push:

jobs:
  cache-job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Cache node modules
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-npm-cache-${{ hashFiles('**/package-lock.json') }}

      - name: Install dependencies
        run: npm install || true
📄 day-44-secrets-artifacts.md
# Day 44 – Secrets, Artifacts & Real CI Tests

---

## 1. GitHub Secrets

### What I learned:
- Secrets are encrypted values stored in GitHub
- Accessed using `${{ secrets.NAME }}`
- Never printed directly in logs

### Why not print secrets?
- Logs can be exposed accidentally
- Security risk if leaked
- GitHub masks them automatically

---

## 2. Using Secrets Safely

- Passed secrets as environment variables
- Used inside shell without exposing value

---

## 3. Artifacts

### What are artifacts?
- Files generated during workflow runs
- Stored and downloadable from GitHub UI

### Use cases:
- Build outputs
- Logs
- Reports

---

## 4. Downloading Artifacts Between Jobs

- Job 1 generates file
- Job 2 downloads and uses it
- Useful for multi-stage pipelines

---

## 5. Real Test CI

### What I did:
- Ran script inside GitHub Actions
- Pipeline failed when script failed
- Fixed script → pipeline turned green

### Key learning:
- CI automatically validates code execution

---

## 6. Caching

### What I learned:
- Cache stores dependencies (like npm modules)
- Speeds up pipeline execution
- Stored on GitHub cache servers

### Benefit:
- Faster builds on repeated runs