## Overview
Today I implemented a complete GitOps pipeline for the AI-BankApp where code changes flow automatically from GitHub to EKS using GitHub Actions + DockerHub + ArgoCD.

---

## GitOps Pipeline Architecture


Developer Code Change
↓
GitHub Push (feat/gitops)
↓
GitHub Actions CI

Build Maven app
Run tests
Build Docker image
Push to DockerHub
Update k8s deployment YAML
Commit back to Git
↓
Git Repository Updated
↓
ArgoCD Sync (polling/webhook)
↓
EKS Cluster Update
Rolling deployment
Zero downtime release
↓
Application Live

---

## GitHub Actions Workflow (Key Steps)

- Checkout code
- Setup JDK 21 + Maven build
- Run tests
- Generate Git SHA image tag
- Build & push Docker image
- Update `k8s/bankapp-deployment.yml`
- Commit updated manifest (`[skip ci]` prevents loop)

---

## Why `[skip ci]` is important
Prevents infinite pipeline loop when CI commits updated Kubernetes manifests.

---

## ArgoCD Role in Pipeline

ArgoCD continuously:
- Watches Git repo (k8s/ folder)
- Detects image tag changes
- Compares desired vs live state
- Performs rolling update in EKS

---

## Drift Detection Tests

### 1. Scaling manually

kubectl scale deployment bankapp --replicas=1

✔ ArgoCD restored desired replica count automatically

### 2. Image tampering

kubectl set image deployment/bankapp bankapp=nginx

✔ ArgoCD reverted to Git-defined image

### 3. Resource deletion

kubectl delete service bankapp-service

✔ ArgoCD recreated service

---

## Key Observations
- Drift detection time: ~2–3 minutes (default sync interval)
- selfHeal ensures cluster always matches Git
- Manual changes do not persist

---

## Full DevOps Flow Connected

Git → GitHub Actions → Docker → Git → ArgoCD → Kubernetes → Observability

---

## Teardown Summary
- Deleted ArgoCD applications
- Destroyed EKS cluster using Terraform
- Confirmed all AWS resources removed

---

## Key Takeaways
- Git is the single source of truth
- CI builds, GitOps deploys
- ArgoCD ensures continuous reconciliation
- Zero manual kubectl deployment required