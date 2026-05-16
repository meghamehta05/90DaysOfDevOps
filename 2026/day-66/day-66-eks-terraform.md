Project Structure
terraform-eks/
  providers.tf
  vpc.tf
  eks.tf
  variables.tf
  outputs.tf
  terraform.tfvars
  k8s/
    nginx-deployment.yaml
 providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }
}

provider "aws" {
  region = var.region
}
 variables.tf
variable "region" {
  type = string
}

variable "cluster_name" {
  type    = string
  default = "terraweek-eks"
}

variable "cluster_version" {
  type    = string
  default = "1.31"
}

variable "node_instance_type" {
  type    = string
  default = "t3.medium"
}

variable "node_desired_count" {
  type    = number
  default = 2
}

variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}
 vpc.tf (EKS-ready VPC)
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "terraweek-vpc"
  cidr = var.vpc_cidr

  azs             = ["${var.region}a", "${var.region}b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true
  enable_dns_hostnames = true

  public_subnet_tags = {
    "kubernetes.io/role/elb" = 1
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = 1
  }
}
 Why EKS needs this?
Public subnets → expose LoadBalancers (internet-facing services)
Private subnets → worker nodes stay secure inside VPC
NAT Gateway → lets private nodes pull images from internet safely
Subnet tags → tells Kubernetes where to place LoadBalancers
 eks.tf
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access = true

  eks_managed_node_groups = {
    terraweek_nodes = {
      ami_type       = "AL2_x86_64"
      instance_types = [var.node_instance_type]

      min_size     = 1
      max_size     = 3
      desired_size = var.node_desired_count
    }
  }

  tags = {
    Environment = "dev"
    Project     = "TerraWeek"
    ManagedBy   = "Terraform"
  }
}
 outputs.tf
output "cluster_name" {
  value = module.eks.cluster_name
}

output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}

output "cluster_region" {
  value = var.region
}
 terraform.tfvars
region = "ap-south-1"
 Commands Run
terraform init
terraform plan
terraform apply
aws eks update-kubeconfig --name terraweek-eks --region ap-south-1
kubectl get nodes
kubectl get pods -A
 Nginx Deployment (k8s/nginx-deployment.yaml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-terraweek
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
kubectl apply -f k8s/nginx-deployment.yaml
kubectl get svc -w
 Verification Outputs
kubectl get nodes → 2 nodes in Ready state
kubectl get pods → nginx pods running
kubectl get svc → LoadBalancer EXTERNAL-IP assigned
 What Terraform Created (~30+ resources)
VPC + Subnets
NAT Gateway + Elastic IP
Route tables
Security groups
IAM roles for EKS
EKS cluster
Managed node group
Auto Scaling Group
CloudWatch logs setup
 Destroy Process
kubectl delete -f k8s/nginx-deployment.yaml
terraform destroy
Verification
No EKS cluster
No EC2 worker nodes
No VPC or NAT Gateway
No Load Balancers
AWS cleaned successfully
 Reflection
Manual Kubernetes (kind/minikube)
Fast setup
Local only
No cloud networking complexity
EKS with Terraform
Production-grade
Highly scalable
Includes IAM, VPC, networking, security
Fully reproducible infrastructure

 Difference:
Local cluster = learning
EKS + Terraform = real-world DevOps engineering

 Final Outcome

 Fully automated AWS EKS cluster
 kubectl connected
 Nginx deployed via LoadBalancer
 Clean destroy with Terraform