1. Objective

Today’s goal is to complete the production-grade Kubernetes workflow by deploying the AI-BankApp on EKS using GitOps (ArgoCD).

You will move from:

Manual kubectl deployments not works
to
Fully automated GitOps deployment via ArgoCD 
2. What You Built So Far
EKS cluster using Terraform
VPC with multi-AZ networking
Managed node groups (t3.medium)
Add-ons (CoreDNS, CNI, EBS CSI, metrics-server)
ArgoCD installed on cluster
AI-BankApp Kubernetes manifests ready
3. GitOps Architecture
GitHub Repo
    |
    | (commit changes)
    v
ArgoCD Controller (EKS Cluster)
    |
    | sync
    v
Kubernetes Cluster
    |
    ├── BankApp (Spring Boot)
    ├── MySQL (EBS-backed PVC)
    ├── Ollama AI Service
    └── HPA + Services + ConfigMaps
4. Step 1: Verify ArgoCD is Running
kubectl get pods -n argocd
kubectl get svc -n argocd

Get password:

kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d

Access UI:

http://<ARGOCD-LB>

Login:

Username: admin
Password: retrieved above
5. Step 2: Create ArgoCD Application

Create file: argocd-app.yaml

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

Apply it:

kubectl apply -f argocd-app.yaml
6. Step 3: Sync Application

ArgoCD will automatically:

Detect repo changes
Apply Kubernetes manifests
Maintain cluster state
Self-heal drift

Check sync status:

argocd app list
argocd app get bankapp
7. Step 4: Verify Deployment
kubectl get pods -n bankapp
kubectl get svc -n bankapp
kubectl get pvc -n bankapp

Expected:

MySQL → Running (EBS attached)
Ollama → Running (model loaded)
BankApp → Running (Spring Boot)
8. Step 5: Access Application

Port forward:

kubectl port-forward svc/bankapp-service -n bankapp 8080:8080

Open:

http://localhost:8080

Test:

User registration
Login flow
AI chatbot (Ollama integration)
9. Step 6: GitOps Workflow Test

Make a change:

echo "replicas: 2" >> k8s/bankapp-deployment.yml
git add .
git commit -m "test: update replicas"
git push origin feat/gitops

ArgoCD will:

Detect change
Sync automatically
Update deployment in cluster
10. Key Concepts Learned
GitOps Principles
Git is single source of truth
Cluster state always matches Git
Changes are auditable and versioned
ArgoCD Features
Auto sync
Self-healing
Drift detection
Rollback support
Why GitOps matters
No manual kubectl in production
Safe deployments
Full rollback history
Team collaboration friendly
11. CI/CD Flow (Final Architecture)
Developer
   |
   v
GitHub Repo
   |
   | CI (GitHub Actions)
   | - Build Docker Image
   | - Push to registry
   | - Update manifest (image tag)
   v
Git Push
   |
   v
ArgoCD
   |
   v
EKS Cluster (AI-BankApp)
12. Benefits Achieved
Zero manual deployments
Fully automated infrastructure
Production-grade Kubernetes setup
Scalable microservice architecture
Observability + persistence ready
13. Cleanup (Optional)
kubectl delete application bankapp -n argocd
kubectl delete namespace bankapp

Or destroy entire infra:

cd terraform
terraform destroy
14. Summary

Today you completed:

GitOps setup with ArgoCD
Automated deployment of AI-BankApp on EKS
Continuous sync between GitHub and Kubernetes
End-to-end production deployment pipeline