Update: .github/workflows/main-pipeline.yml (ADD SECURITY STEP)

 Add this after Docker build, before deploy

name: Scan Docker Image for Vulnerabilities
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: myapp:latest
    format: 'table'
    exit-code: '1'
    severity: 'CRITICAL,HIGH'
Update: PR Pipeline (Dependency Security Check)

 Add this job or step inside pr-pipeline.yml

name: Check Dependencies for Vulnerabilities
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: critical

 Must be under pull_request trigger only.

 Add Permissions (IMPORTANT SECURITY HARDENING)
Update ALL major workflows (PR + main)

Example:

permissions:
  contents: read

For PR workflows that may comment:

permissions:
  contents: read
  pull-requests: write
 Optional Upgrade: Full Secure Main Pipeline

Here is how your secure main pipeline flow now looks:

name: Secure Main Pipeline

on:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    uses: ./.github/workflows/reusable-build-test.yml

  docker:
    needs: test
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image_name: myapp
      tag: latest
    secrets:
      docker_username: ${{ secrets.DOCKER_USERNAME }}
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  security-scan:
    needs: docker
    runs-on: ubuntu-latest

    steps:
      - name: Scan Docker Image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:latest
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'

  deploy:
    needs: security-scan
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Deploy
        run: echo "Deploying secure image myapp:latest "
📄 day-49-devsecops.md
# Day 49 – DevSecOps

## What is DevSecOps?

DevSecOps means integrating security checks directly into CI/CD pipelines so vulnerabilities are detected early, before deployment.

Instead of fixing issues after release, we prevent them during development using automation.

---

## Key Things I Implemented

- Docker image vulnerability scanning (Trivy)
- Dependency vulnerability scanning
- GitHub secret scanning
- Workflow permissions restriction

---

## Security Flow Added

PR Stage:
- Dependency review → blocks vulnerable packages

Main Stage:
- Docker build
- Trivy scan (fails on HIGH/CRITICAL)
- Deploy only if secure

---

## What I Learned

- Security must be automated, not manual
- CI/CD pipelines can enforce security rules
- Early detection reduces production risk

---

## Pipeline Security Diagram

PR → Build → Dependency Scan → Merge

Main → Build → Docker → Trivy Scan → Deploy


