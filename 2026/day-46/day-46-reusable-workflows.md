.github/workflows/reusable-build.yml
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      app_name:
        required: true
        type: string
      environment:
        required: true
        type: string
        default: staging

    secrets:
      docker_token:
        required: true

    outputs:
      build_version:
        description: "Generated build version"
        value: ${{ jobs.build.outputs.build_version }}

jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      build_version: ${{ steps.version.outputs.build_version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Print build info
        run: |
          echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

      - name: Check secret
        run: echo "Docker token is set: ${{ secrets.docker_token != '' }}"

      - name: Generate version
        id: version
        run: |
          SHORT_SHA=$(echo ${{ github.sha }} | cut -c1-7)
          echo "build_version=v1.0-$SHORT_SHA" >> $GITHUB_OUTPUT
📄 .github/workflows/call-build.yml
name: Call Reusable Build Workflow

on:
  push:
    branches: [main]

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      app_name: "my-web-app"
      environment: "production"
    secrets:
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  print-version:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Show build version
        run: echo "Build version: ${{ needs.build.outputs.build_version }}"
📁 Composite Action
.github/actions/setup-and-greet/action.yml
name: Setup and Greet Action

description: Custom composite action to greet user

inputs:
  name:
    required: true
    type: string
  language:
    required: false
    default: en

outputs:
  greeted:
    description: "Whether greeting was done"
    value: ${{ steps.greet.outputs.greeted }}

runs:
  using: "composite"

  steps:
    - name: Greeting step
      id: greet
      shell: bash
      run: |
        if [ "${{ inputs.language }}" == "en" ]; then
          echo "Hello ${{ inputs.name }}!"
        else
          echo "Namaste ${{ inputs.name }}!"
        fi

        echo "Date: $(date)"
        echo "OS: $RUNNER_OS"

        echo "greeted=true" >> $GITHUB_OUTPUT
📄 Example workflow using composite action
name: Use Composite Action

on:
  push:

jobs:
  greet:
    runs-on: ubuntu-latest

    steps:
      - name: Use custom action
        uses: ./.github/actions/setup-and-greet
        with:
          name: "Megha"
          language: "en"
📄 day-46-reusable-workflows.md
# Day 46 – Reusable Workflows & Composite Actions

---

## 1. What I Learned

### Reusable Workflow
- Created using `workflow_call`
- Works like a function
- Can be called from other workflows

### Why it matters:
- Removes duplication
- Standardizes CI/CD across projects

---

## 2. Composite Action

- Custom GitHub Action made from multiple steps
- Reusable inside workflows using `uses:`

---

## 3. Reusable vs Composite

| Feature | Reusable Workflow | Composite Action |
|---|---|---|
| Triggered by | workflow_call | uses in step |
| Can contain jobs? | Yes | No |
| Multiple steps? | Yes | Yes |
| Location | .github/workflows | .github/actions |
| Accept secrets | Yes | Limited |
| Best for | Full pipelines | Small reusable logic |

---

## 4. Outputs

- Reusable workflows can pass outputs between jobs
- Used for versioning and build tracking

---

## 5. Key Learning

- Workflows can behave like functions
- Actions can behave like reusable utilities
- GitHub Actions is modular and scalable