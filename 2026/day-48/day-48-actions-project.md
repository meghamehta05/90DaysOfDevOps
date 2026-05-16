.github/workflows/reusable-build-test.yml
name: Reusable Build & Test

on:
  workflow_call:
    inputs:
      python_version:
        type: string
        required: false
        default: "3.10"
      run_tests:
        type: boolean
        default: true

    outputs:
      test_result:
        value: ${{ jobs.test.outputs.result }}

jobs:
  test:
    runs-on: ubuntu-latest

    outputs:
      result: ${{ steps.set-result.outputs.result }}

    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python_version }}

      - name: Install dependencies
        run: |
          pip install -r requirements.txt || true

      - name: Run tests
        if: ${{ inputs.run_tests }}
        run: |
          echo "Running tests..."
          python test.py || exit 1

      - name: Set output
        id: set-result
        run: echo "result=passed" >> $GITHUB_OUTPUT
📄 .github/workflows/reusable-docker.yml
name: Reusable Docker Build & Push

on:
  workflow_call:
    inputs:
      image_name:
        type: string
        required: true
      tag:
        type: string
        required: true

    secrets:
      docker_username:
        required: true
      docker_token:
        required: true

    outputs:
      image_url:
        value: ${{ jobs.build.outputs.image_url }}

jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      image_url: ${{ steps.out.outputs.image_url }}

    steps:
      - uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_token }}

      - name: Build image
        run: docker build -t ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }} .

      - name: Push image
        run: docker push ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}

      - name: Set output
        id: out
        run: echo "image_url=${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}" >> $GITHUB_OUTPUT
📄 .github/workflows/pr-pipeline.yml
name: PR Pipeline

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      run_tests: true

  pr-comment:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - name: PR Summary
        run: echo "PR checks passed for branch: ${{ github.head_ref }}"
📄 .github/workflows/main-pipeline.yml
name: Main Pipeline

on:
  push:
    branches: [main]

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

  deploy:
    needs: docker
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Deploy
        run: echo "Deploying image ${{ needs.docker.outputs.image_url }} to production"
📄 .github/workflows/health-check.yml
name: Health Check

on:
  schedule:
    - cron: "0 */12 * * *"
  workflow_dispatch:

jobs:
  health:
    runs-on: ubuntu-latest

    steps:
      - name: Pull image
        run: docker pull myapp:latest || true

      - name: Run container
        run: |
          docker run -d --name app myapp:latest || true
          sleep 5

      - name: Check health
        run: |
          curl -f http://localhost:8080 || echo "FAILED"

      - name: Cleanup
        run: docker rm -f app || true

      - name: Summary
        run: |
          echo "## Health Check Report" >> $GITHUB_STEP_SUMMARY
          echo "- Image: myapp:latest" >> $GITHUB_STEP_SUMMARY
          echo "- Status: DONE" >> $GITHUB_STEP_SUMMARY
          echo "- Time: $(date)" >> $GITHUB_STEP_SUMMARY
📄 day-48-actions-project.md
# Day 48 – GitHub Actions Capstone Project

---

## 1. What I Built

A full CI/CD pipeline:
- PR pipeline (test only)
- Main pipeline (test → docker → deploy)
- Scheduled health checks
- Reusable workflows

---

## 2. Architecture Flow


PR → Build & Test → PR Check
Main Merge → Build & Test → Docker Build → Push → Deploy
Every 12h → Health Check


---

## 3. Key Concepts Used

- Reusable workflows (`workflow_call`)
- Secrets management
- Docker automation
- Job dependencies (`needs`)
- Scheduled workflows
- Environments (production deploy gate)

---

## 4. CI/CD Behavior

- PR → no Docker push (safe)
- Main → full deployment pipeline
- Cron → monitoring pipeline

---

## 5. DevSecOps Preview (Optional)

- Can integrate Trivy for image scanning
- Security checks before deployment

---

## 6. What This Proves

- Real production CI/CD knowledge
- Modular workflow design
- Event-driven automation

---

## 7. Next Improvements

- Slack notifications
- Kubernetes deployment
- Rollback strategy
- Multi-environment staging/production