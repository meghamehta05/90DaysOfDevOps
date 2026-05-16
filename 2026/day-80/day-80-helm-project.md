1. Introduction

Today I completed the final Helm milestone by building a **multi-environment deployment system** for the AI-BankApp using a single Helm chart.

The system now supports:
- Dev environment
- Staging environment
- Production environment
- CI/CD integration
- Helm hooks + tests
- Packaged chart distribution

---

## 2. Environment Strategy

One Helm chart → three environments.

:contentReference[oaicite:0]{index=0} enables environment-specific configuration using `values.yaml`.

---

## 3. Environment Comparison

| Feature | Dev | Staging | Production |
|--------|-----|---------|------------|
| Replicas | 1 | 2–3 | 2–4 |
| Image tag | latest | v1.2.0 | v1.2.0 |
| MySQL storage | 2Gi | 5Gi | 20Gi |
| Memory usage | Low | Medium | High |


---

## 4. Values Files Overview

### values-dev.yaml
- Minimal resources
- autoscaling disabled
- fast iteration setup

### values-staging.yaml
- Balanced configuration
- HPA enabled
- production-like setup

### values-prod.yaml
- High availability
- strong resource limits
- gateway enabled
- production-grade scaling

---

## 5. Helm Hook Implementation

A pre-install job ensures database readiness before deployment.

```yaml id="hook1"
annotations:
  "helm.sh/hook": pre-install,pre-upgrade
  "helm.sh/hook-weight": "0"
  "helm.sh/hook-delete-policy": before-hook-creation
Purpose:
Ensures MySQL is running before app starts
Prevents deployment failures
Works during install and upgrade
6. Helm Test

A test pod validates application health:

annotations:
  "helm.sh/hook": test

Test command:

helm test bankapp-dev -n dev
Result:
Calls /actuator/health
Confirms Spring Boot is running
7. Helm Package Management
Package chart:
helm package bankapp/

Generates:

bankapp-0.1.0.tgz
Version upgrade:
version: 0.2.0
appVersion: 1.1.0
8. Deployment Methods
Dev deployment:
helm install bankapp-dev bankapp/ -f values-dev.yaml
Render staging:
helm template bankapp-staging bankapp/ -f values-staging.yaml
Render production:
helm template bankapp-prod bankapp/ -f values-prod.yaml
9. CI/CD Integration

Helm integrates into GitOps pipelines by updating values instead of raw YAML.

CI workflow logic:
- name: Update image tag
  run: |
    yq -i '.bankapp.image.tag = "${{ github.sha }}"' values-prod.yaml
Flow:
Code push → Build image → Update values.yaml → ArgoCD sync → Helm deploy
10. ArgoCD + Helm Advantage

Instead of raw manifests:

Feature	Raw YAML	Helm
Updates	Manual	Automated
Environments	Duplicate files	Values files
Rollbacks	Manual	Built-in

11. Production Best Practices
Use helm upgrade --install
Always use --atomic
Use helm diff before upgrades
Never store real secrets in values.yaml
Use external secret managers in production
12. Key Learnings
Helm enables full environment abstraction
Hooks provide lifecycle control
Charts can be packaged and shared
CI/CD becomes simpler with values-based updates
GitOps works best with Helm + ArgoCD