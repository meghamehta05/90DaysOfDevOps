What is GitOps?

GitOps is a deployment model where:
- Git is the single source of truth for infrastructure and application state
- Any change in infrastructure is made through Git commits
- A controller (ArgoCD) continuously reconciles the cluster state with Git

If someone changes resources manually in the cluster, GitOps tools detect and correct it automatically (self-healing).

---

## GitOps vs Traditional CI/CD

| Feature | Traditional CI/CD | GitOps |
|--------|------------------|--------|
| Deployment | CI runs kubectl apply | Git commit triggers sync |
| Source of truth | CI pipeline | Git repository |
| Drift detection | Not available | Continuous reconciliation |
| Rollback | Manual pipeline rollback | git revert |
| Audit | CI logs | Git history |
| Security | CI has cluster access | Only ArgoCD accesses cluster |

---

## AI-BankApp GitOps Flow


Developer push → GitHub Actions CI
→ Build + Test
→ Docker image build & push
→ Update image tag in k8s manifests
→ Commit back to Git

ArgoCD:
→ Detects Git change
→ Compares cluster state
→ Applies changes automatically
→ Ensures cluster matches Git


---

## GitOps Principles

1. Declarative
   - Infrastructure defined using YAML

2. Versioned & Immutable
   - Everything stored in Git

3. Pulled Automatically
   - ArgoCD pulls changes (not pushed by CI)

4. Continuously Reconciled
   - Drift is detected and corrected automatically

---

## ArgoCD Status Check

```bash
kubectl get pods -n argocd

Get admin password:

kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d

Access UI:

kubectl port-forward svc/argocd-server -n argocd 8443:443

Login:

Username: admin
Password: retrieved above

---

## ArgoCD Application Manifest Explained

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bankapp
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/<your-username>/AI-BankApp-DevOps.git
    targetRevision: feat/gitops
    path: k8s

  destination:
    server: https://kubernetes.default.svc
    namespace: bankapp

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - ServerSideApply=true
Key Fields:
repoURL → Git repository source
targetRevision → branch to track
path → Kubernetes manifests folder
prune → deletes removed resources from cluster
selfHeal → fixes manual drift automatically
ServerSideApply → avoids conflicts in updates
Self-Healing Test Results
Test 1: Scale Deployment manually
kubectl scale deployment bankapp -n bankapp --replicas=1

Result:

ArgoCD reverted replicas back to Git-defined value
Drift corrected automatically within minutes
Test 2: Delete ConfigMap
kubectl delete configmap bankapp-config -n bankapp

Result:

ConfigMap was recreated automatically by ArgoCD
Test 3: Modify ConfigMap manually
kubectl edit configmap bankapp-config -n bankapp

Result:

Changes were overwritten by Git state
Key Learnings
Git is the only source of truth
Kubernetes cluster is fully declarative
Manual kubectl changes are temporary
ArgoCD ensures continuous reconciliation
Conclusion

ArgoCD removes manual deployment risk and enforces:

consistency
auditability
automation
self-healing infrastructure

---

##  Push Commands (copy exactly)

Run this inside your repo root:

```bash
mkdir -p 2026/day-84

# create file
nano 2026/day-84/day-84-gitops-argocd.md


