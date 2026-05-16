### 1. Automated Sync (Default in AI-BankApp)
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
Sync happens automatically after Git changes
Self-heals drift automatically
Best for dev/staging environments
2. Manual Sync (Production Mode)
argocd app set bankapp --sync-policy none
Detects drift but does NOT apply changes automatically
Requires manual approval
Safer for production systems
Manual Sync Commands
argocd app diff bankapp
argocd app sync bankapp --dry-run
argocd app sync bankapp
Sync Waves (Resource Ordering)
Why Sync Waves?

AI-BankApp has dependencies:

DB must start before app
Config must exist before deployment
Deployment Order
Wave	Resources
-2	Namespace, StorageClass
-1	PVC, ConfigMap, Secrets
0	MySQL, Ollama, Services
1	BankApp Deployment
2	HPA
Example Annotation
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"
ArgoCD Rollbacks
View History
argocd app history bankapp
Rollback via CLI
argocd app rollback bankapp 1
GitOps Correct Rollback



git revert HEAD
git push origin feat/gitops
Difference
Type	Effect
ArgoCD rollback	Cluster-only change
git revert	True source-of-truth correction
App of Apps Pattern
Concept

One parent app manages multiple child apps.

Root Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/<your-username>/AI-BankApp-DevOps.git
    targetRevision: feat/gitops
    path: argocd-apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
Child Apps
bankapp
monitoring
envoy-gateway
Architecture
root-app
   ├── bankapp
   ├── monitoring
   └── envoy-gateway
ArgoCD Notifications
Event Triggers
Sync succeeded
Sync failed
Health degraded
ConfigMap Example
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  trigger.on-sync-succeeded: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-sync-succeeded]

  template.app-sync-succeeded: |
    message: "Sync successful: {{.app.metadata.name}}"
ArgoCD Projects (Multi-Tenancy)
Create Project
argocd proj create bankapp-team \
--description "BankApp team project" \
--src https://github.com/<your-username>/AI-BankApp-DevOps.git \
--dest https://kubernetes.default.svc,bankapp
Why Projects?
Restrict repo access
Restrict namespace access
Prevent cross-team damage
RBAC Example
policy.csv: |
  p, role:dev, applications, sync, bankapp-team/*, allow
  p, role:dev, applications, rollback, bankapp-team/*, deny
Key Learnings
Sync waves control deployment order
Rollback ≠ Git revert (GitOps rule)
App of Apps manages large-scale systems
Notifications add observability
Projects + RBAC enforce multi-team security
Conclusion

ArgoCD is not just a deployment tool — it is a full production control system for Kubernetes.



Then:

git add 2026/day-85/day-85-argocd-deep-dive.md
git commit -m "Day 85: ArgoCD deep dive - sync waves, rollback, app of apps"
git push origin main