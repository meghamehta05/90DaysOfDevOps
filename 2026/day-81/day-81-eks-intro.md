## 1. Introduction

Today I moved from local Kubernetes (Kind) to a production-grade managed cluster using  
:contentReference[oaicite:0]{index=0} provisioned using Terraform.

This marks the beginning of real cloud-native infrastructure practice.



## 2. What is Managed Kubernetes?

In a managed Kubernetes system:

### AWS manages:
- Control plane (API server, scheduler, etcd)
- High availability across multiple availability zones
- Security patching and upgrades

### We manage:
- Worker nodes (EC2 instances)
- Applications (pods, services)
- Storage, scaling, and deployments

---

## 3. EKS Architecture Overview

### 1. Control Plane
- Fully managed by AWS
- Runs outside your VPC
- Handles cluster state and API requests

### 2. Worker Nodes (Node Groups)
- EC2 instances running pods
- AI-BankApp uses managed node groups (t3.medium)

### 3. Networking (VPC)
- Public subnets → Load Balancers
- Private subnets → Worker nodes
- Intra subnets → control plane communication

### 4. Add-ons used in AI-BankApp

- CoreDNS → service discovery  
- kube-proxy → networking  
- VPC CNI → pod IP assignment  
- Metrics Server → HPA support  
- EBS CSI Driver → persistent storage  
- Pod Identity Agent → IAM integration  

---

## 4. Terraform Architecture (AI-BankApp)

| File | Purpose |
|------|--------|
| provider.tf | AWS + Helm providers |
| vpc.tf | VPC with multi-AZ subnets |
| eks.tf | EKS cluster + node groups |
| argocd.tf | Installs ArgoCD via Helm |
| variables.tf | Input variables |
| terraform.tfvars | Default config |
| outputs.tf | kubeconfig + commands |

---

## 5. VPC Design

- 3 Availability Zones
- Public subnets → LoadBalancers
- Private subnets → Worker nodes
- Intra subnets → internal control traffic
- NAT Gateway → internet access for private nodes

---

## 6. EKS Cluster Configuration

- Kubernetes version: 1.35  
- Node type: t3.medium  
- Desired nodes: 3  
- Max nodes: 5  
- Managed node group enabled  
- Amazon Linux 2023 AMI  

---

## 7. Terraform Deployment Flow

```bash id="tf81"
terraform init
terraform plan
terraform apply

This provisions:

VPC
EKS cluster
Node groups
IAM roles
Add-ons
ArgoCD installation

---

## 8. Connect to Cluster

```bash id="kub81"
aws eks update-kubeconfig --name bankapp-eks --region us-west-2

Verify:

kubectl get nodes
kubectl cluster-info

Expected:

3 nodes in Ready state
Spread across AZs
Instance type: t3.medium
9. System Components
kubectl get pods -n kube-system

Includes:

CoreDNS
Metrics Server
AWS VPC CNI
EBS CSI Driver
10. AI-BankApp Deployment

Deployed using raw manifests:

kubectl apply -f k8s/

Order:

Namespace
ConfigMap + Secrets
MySQL
Ollama
BankApp
HPA
11. Persistent Storage

MySQL and Ollama use EBS volumes:

Amazon EBS

kubectl get pvc -n bankapp
MySQL → 5Gi
Ollama → 10Gi
12. ArgoCD Status
Installed via Terraform
Exposed via LoadBalancer
Used later for GitOps deployment
13. Cost Breakdown
Component	Monthly Cost
EKS Control Plane	~$73
3x t3.medium nodes	~$91
NAT Gateway	~$33
EBS storage	~$1–2
LoadBalancer	~$18
Total	~$220/month
14. Clean Up

To remove workloads:

kubectl delete -f k8s/

To destroy full infra:

terraform destroy
15. Key Learnings
EKS = managed control plane + self-managed nodes
Terraform automates full AWS infrastructure
VPC CNI gives pods real VPC IPs
EBS CSI enables persistent storage
ArgoCD prepares GitOps pipeline
16. Conclusion

Today I successfully:

Understood EKS architecture
Studied Terraform-based infrastructure
Connected kubectl to AWS EKS
Deployed AI-BankApp on a real cloud cluster

This completes the shift from local Kubernetes → production-grade AWS Kubernetes.