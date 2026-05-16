.github/workflows/multi-os.yml
name: Multi OS Runners

on:
  push:

jobs:
  ubuntu-job:
    runs-on: ubuntu-latest
    steps:
      - name: OS Info
        run: |
          echo "OS: Ubuntu"
          hostname
          whoami

  windows-job:
    runs-on: windows-latest
    steps:
      - name: OS Info
        run: |
          echo "OS: Windows"
          hostname

  mac-job:
    runs-on: macos-latest
    steps:
      - name: OS Info
        run: |
          echo "OS: macOS"
          hostname
          whoami
 .github/workflows/ubuntu-tools.yml
name: Ubuntu Tools Check

on:
  push:

jobs:
  tools:
    runs-on: ubuntu-latest

    steps:
      - name: Check tools versions
        run: |
          echo "Docker:"
          docker --version || true

          echo "Python:"
          python3 --version

          echo "Node:"
          node --version || true

          echo "Git:"
          git --version
 .github/workflows/self-hosted.yml
name: Self Hosted Runner Demo

on:
  push:

jobs:
  self-hosted-job:
    runs-on: self-hosted

    steps:
      - name: Show machine info
        run: |
          hostname
          whoami
          pwd

      - name: Create file
        run: echo "Hello from self-hosted runner" > runner.txt

      - name: Verify file
        run: ls -la

 day-42-runners.md
# Day 42 – GitHub Runners (Hosted vs Self-Hosted)

---

## 1. GitHub-Hosted Runners

### What they are:
- Machines provided and managed by GitHub
- Automatically provisioned for every job

### Who manages them?
- GitHub fully manages OS, updates, scaling, and security

---

## 2. Pre-installed Tools (Ubuntu Runner)

Examples:
- Docker
- Python
- Node.js
- Git

### Why it matters:
- Saves setup time
- Ensures consistent CI environment
- No manual configuration needed

---

## 3. Self-Hosted Runner

### What I did:
- Installed GitHub runner on my own machine/VM
- Registered it with my repository
- Verified it shows as Idle in GitHub

---

## 4. Self-Hosted Workflow

- Runs on my own hardware
- Created file `runner.txt`
- Verified file exists locally after pipeline run

---

## 5. Labels

- Labels help target specific runners
- Example:
  - `self-hosted`
  - `linux`
  - `my-runner`

### Why useful:
- When multiple runners exist
- Helps route jobs correctly

---

## 6. Comparison Table

| Feature | GitHub-Hosted | Self-Hosted |
|--------|--------------|-------------|
| Who manages it? | GitHub | User |
| Cost | Free (limited minutes) | Your own resources |
| Pre-installed tools | Yes | You configure |
| Best for | General CI/CD | Custom workloads |
| Security | Safer (isolated) | User-managed security |

---

## Final Learning

- Runners execute CI/CD jobs
- GitHub-hosted = easy & scalable
- Self-hosted = powerful & customizable
- Real DevOps uses both depending on need
 Now Push Everything