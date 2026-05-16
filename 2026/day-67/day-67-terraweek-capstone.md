1. Project Structure
terraweek-capstone/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── locals.tf
│
├── dev.tfvars
├── staging.tfvars
├── prod.tfvars
│
├── .gitignore
│
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    ├── security-group/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── ec2-instance/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
 2. Workspace Answers (Important Theory)
1. What does terraform.workspace return?

It returns the current active workspace name:

default / dev / staging / prod
2. Where is state stored?

Locally:

terraform.tfstate.d/<workspace>/terraform.tfstate
3. Workspaces vs separate folders
Workspaces	Separate Folders
Single codebase	Duplicate code
Easy switching	Hard maintenance
Shared config	Fully isolated
Faster scaling	More manual work

 Best for same infrastructure, multiple environments

 3. Providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
 4. locals.tf (Workspace Awareness)
locals {
  environment = terraform.workspace
  name_prefix  = "${var.project_name}-${local.environment}"

  common_tags = {
    Project     = var.project_name
    Environment = local.environment
    ManagedBy   = "Terraform"
    Workspace   = terraform.workspace
  }
}
 5. variables.tf
variable "project_name" {
  default = "terraweek"
}

variable "vpc_cidr" {}

variable "subnet_cidr" {}

variable "instance_type" {}

variable "ingress_ports" {
  type = list(number)
}
 6. MODULE 1 — VPC
modules/vpc/main.tf
resource "aws_vpc" "this" {
  cidr_block = var.cidr

  tags = merge(var.tags, {
    Name = "${var.environment}-vpc"
  })
}

resource "aws_subnet" "this" {
  vpc_id     = aws_vpc.this.id
  cidr_block = var.public_subnet_cidr

  tags = merge(var.tags, {
    Name = "${var.environment}-subnet"
  })
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.this.id

  tags = var.tags
}
outputs.tf
output "vpc_id" {
  value = aws_vpc.this.id
}

output "subnet_id" {
  value = aws_subnet.this.id
}
 7. MODULE 2 — SECURITY GROUP
main.tf
resource "aws_security_group" "this" {
  name   = "${var.environment}-sg"
  vpc_id = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = var.tags
}
 8. MODULE 3 — EC2 INSTANCE
main.tf
resource "aws_instance" "this" {
  ami                    = "ami-0c2af51e265bd5e0e"
  instance_type         = var.instance_type
  subnet_id             = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = merge(var.tags, {
    Name = "${var.environment}-server"
  })
}
 9. ROOT main.tf (VERY IMPORTANT)
module "vpc" {
  source            = "./modules/vpc"
  cidr              = var.vpc_cidr
  public_subnet_cidr = var.subnet_cidr
  environment       = local.environment
  project_name      = var.project_name
  tags              = local.common_tags
}

module "sg" {
  source         = "./modules/security-group"
  vpc_id         = module.vpc.vpc_id
  ingress_ports  = var.ingress_ports
  environment    = local.environment
  project_name   = var.project_name
  tags           = local.common_tags
}

module "ec2" {
  source              = "./modules/ec2-instance"
  instance_type       = var.instance_type
  subnet_id           = module.vpc.subnet_id
  security_group_ids  = [module.sg.sg_id]
  environment         = local.environment
  project_name        = var.project_name
  tags                = local.common_tags
}
10. ENV FILES
dev.tfvars
vpc_cidr      = "10.0.0.0/16"
subnet_cidr   = "10.0.1.0/24"
instance_type = "t2.micro"
ingress_ports = [22, 80]
staging.tfvars
vpc_cidr      = "10.1.0.0/16"
subnet_cidr   = "10.1.1.0/24"
instance_type = "t2.small"
ingress_ports = [22, 80, 443]
prod.tfvars
vpc_cidr      = "10.2.0.0/16"
subnet_cidr   = "10.2.1.0/24"
instance_type = "t3.small"
ingress_ports = [80, 443]
 11. DEPLOYMENT FLOW
terraform init
Dev
terraform workspace new dev
terraform workspace select dev
terraform apply -var-file="dev.tfvars"
Staging
terraform workspace new staging
terraform workspace select staging
terraform apply -var-file="staging.tfvars"
Prod
terraform workspace new prod
terraform workspace select prod
terraform apply -var-file="prod.tfvars"
 12. VERIFY
terraform workspace select dev
terraform output

Repeat for staging + prod.

Check AWS:

3 VPCs 
3 EC2 instances 
Different CIDRs 
Different instance types 
 13. DESTROY FLOW
terraform workspace select prod
terraform destroy -var-file="prod.tfvars"

terraform workspace select staging
terraform destroy -var-file="staging.tfvars"

terraform workspace select dev
terraform destroy -var-file="dev.tfvars"

Then:

terraform workspace select default
terraform workspace delete dev
terraform workspace delete staging
terraform workspace delete prod