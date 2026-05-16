Day 81 – EKS Intro (Markdown File Content)

Create this file: 2026/day-81/day-81-eks-intro.md

Day 81 – Introduction to Amazon EKS with Terraform
1. EKS Architecture Overview

Amazon EKS (Elastic Kubernetes Service) is AWS’s managed Kubernetes platform. It separates Kubernetes into two main layers:

Control Plane (Managed by AWS)
Kubernetes API Server
etcd (cluster state storage)
Scheduler
Controller Manager
Fully managed, highly available across multiple AZs
AWS handles patching, upgrades, and scaling
Data Plane (Managed by User)
EC2 worker nodes (node groups)
Runs actual application pods
You control instance type, scaling, and capacity
2. What is “Managed Kubernetes”?

Managed Kubernetes means:

AWS runs and maintains the control plane
You only manage worker nodes and workloads
No need to install Kubernetes master components manually
Built-in HA, security, and upgrades
3. EKS Core Components
1. EKS Cluster

Central Kubernetes control plane managed by AWS.

2. Node Groups
Managed Node Groups (used in AI-BankApp)
AWS handles provisioning and upgrades
Auto-scaling supported
Self-managed nodes
User manages EC2 lifecycle manually
Fargate
Serverless pods (no EC2 nodes)
3. VPC Networking
Cluster runs inside a custom VPC
Subnets:
Public → LoadBalancers
Private → Worker nodes
Intra → Control plane networking
4. IAM Integration (IRSA)
IAM Roles for Service Accounts
Allows pods to securely access AWS services
4. EKS Add-ons Used in AI-BankApp

From Terraform configuration:

CoreDNS → Internal DNS resolution
kube-proxy → Service networking
VPC CNI → Assigns VPC IPs to pods
EBS CSI Driver → Enables persistent storage (MySQL, Ollama)
Metrics Server → Enables HPA and kubectl top
Pod Identity Agent → IAM access for pods
5. AI-BankApp Terraform Architecture
Terraform Files Overview
vpc.tf
Creates VPC using terraform AWS module
3 AZs with:
Public subnets (LoadBalancers)
Private subnets (Worker nodes)
Intra subnets (EKS control plane)
NAT Gateway for outbound traffic
eks.tf
Creates EKS cluster (v1.35)
Managed node group:
t3.medium (3–5 nodes)
Installs add-ons:
CoreDNS
VPC CNI
kube-proxy
EBS CSI driver
metrics-server
Enables IRSA (IAM for pods)
argocd.tf
Installs ArgoCD via Helm
Exposes ArgoCD using LoadBalancer service
Depends on EKS cluster
variables.tf

Defines inputs like:

Cluster name
Region
Node size
Scaling limits
terraform.tfvars

Default values:

Region: us-west-2
Cluster: bankapp-eks
Nodes: 3 (min), 5 (max)
Instance type: t3.medium
outputs.tf

Provides:

kubeconfig update command
ArgoCD access details
Cluster endpoint info
6. EKS Architecture Diagram
                AWS Cloud
────────────────────────────────
        EKS Control Plane
   (API Server + etcd + Scheduler)
────────────────────────────────

VPC (bankapp-eks-vpc)
│
├── Public Subnets (AZ1, AZ2, AZ3)
│     └── Load Balancers (ArgoCD, App)
│
├── Private Subnets (AZ1, AZ2, AZ3)
│     └── Worker Nodes (t3.medium)
│           ├── BankApp Pods
│           ├── MySQL Pod
│           └── Ollama Pod
│
└── Intra Subnets
      └── EKS Control Networking
7. Provisioning Flow
Terraform creates VPC
EKS cluster control plane is provisioned
Managed node group joins cluster
AWS installs core add-ons
ArgoCD is deployed via Helm
kubeconfig is updated to access cluster
8. Connect to Cluster
aws eks update-kubeconfig --name bankapp-eks --region us-west-2

Verify:

kubectl get nodes -o wide
kubectl cluster-info

Expected:

3 nodes (t3.medium)
Status: Ready
Spread across AZs
9. System Add-ons Check
kubectl get pods -n kube-system
kubectl get daemonsets -n kube-system
kubectl top nodes
10. AI-BankApp Deployment Validation
kubectl get pods -n bankapp
kubectl get svc -n bankapp
kubectl get pvc -n bankapp
11. ArgoCD Access

Get password:

kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d

Get URL:

kubectl get svc -n argocd argocd-server

Login:

Username: admin
Password: retrieved above
12. Cost Breakdown (EKS Lab)
Component	Monthly Cost
EKS Control Plane	~$73
3x t3.medium nodes	~$91
NAT Gateway	~$33
EBS Storage	~$1–2
LoadBalancer	~$18
Total	~$220/month
Why NAT Gateway is expensive?
Charged hourly + per GB data transfer
Required for private subnet internet access
Always running = always billing
13. Key Learnings
EKS = managed control plane + user-managed nodes
Terraform automates full infrastructure lifecycle
Add-ons are essential for production clusters
IRSA enables secure AWS integration
Multi-AZ design improves availability
14. Cleanup Commands
kubectl delete namespace bankapp
terraform destroy
15. Summary

Today’s focus:

Understood EKS architecture
Explored Terraform-based cluster provisioning
Connected kubectl to AWS EKS
Validated real production-grade Kubernetes setup