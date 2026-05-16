STEP 1: Project Setup
mkdir terraform-aws-infra
cd terraform-aws-infra
🔹 STEP 2: Create providers.tf
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
▶ Initialize Terraform
terraform init
 What ~> 5.0 means:
Allows updates like 5.1, 5.2
Blocks 6.0 (major upgrade not allowed)
🔹 STEP 3: Create VPC Stack (main.tf)
# ---------------- VPC ----------------
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "TerraWeek-VPC"
  }
}

# ---------------- SUBNET ----------------
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true

  tags = {
    Name = "TerraWeek-Public-Subnet"
  }
}

# ---------------- INTERNET GATEWAY ----------------
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "TerraWeek-IGW"
  }
}

# ---------------- ROUTE TABLE ----------------
resource "aws_route_table" "rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "TerraWeek-RT"
  }
}

# ---------------- ROUTE ASSOCIATION ----------------
resource "aws_route_table_association" "rta" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.rt.id
}
▶ Apply
terraform plan
terraform apply
🔹 STEP 4: Security Group + EC2

Add to main.tf:

# ---------------- SECURITY GROUP ----------------
resource "aws_security_group" "sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "TerraWeek-SG"
  }
}

# ---------------- EC2 INSTANCE ----------------
resource "aws_instance" "server" {
  ami                         = "ami-0f5ee92e2d63afc18"
  instance_type               = "t2.micro"
  subnet_id                  = aws_subnet.public.id
  vpc_security_group_ids     = [aws_security_group.sg.id]
  associate_public_ip_address = true

  tags = {
    Name = "TerraWeek-Server"
  }

  lifecycle {
    create_before_destroy = true
  }
}
▶ Apply again
terraform plan
terraform apply
🔹 STEP 5: Explicit Dependency (S3 Bucket)
resource "aws_s3_bucket" "logs" {
  bucket = "terraweek-logs-2026-unique-12345"

  depends_on = [aws_instance.server]
}
🔹 STEP 6: Dependency Graph
terraform graph

OR export:

terraform graph | dot -Tpng > graph.png
🔹 STEP 7: Understand Dependencies
✔ Implicit dependencies:
subnet → VPC (aws_vpc.main.id)
IGW → VPC
route table → IGW
EC2 → subnet + SG

Terraform automatically figures this out.

✔ Explicit dependency:
depends_on = [aws_instance.server]

Used when:

no direct reference exists
but order still matters
🔹 STEP 8: Lifecycle Rules
create_before_destroy
lifecycle {
  create_before_destroy = true
}

✔ New resource created before deleting old one

ignore_changes

Used when:

manual AWS changes should NOT be overwritten
prevent_destroy

Used for:

production DB
critical infrastructure
🔹 STEP 9: Destroy Everything
terraform destroy


Take:

terraform init
terraform apply


AWS Console:

VPC
Subnet
IGW
EC2 instance
terraform graph

Create:

day-62-providers-resources.md
✔ Include:
1. What is provider?

Plugin that connects Terraform to AWS.

2. Resources created:
VPC
Subnet
Internet Gateway
Route Table
Security Group
EC2 Instance
3. Implicit vs Explicit dependency
Implicit: Terraform auto detects
Explicit: manually defined using depends_on
4. Lifecycle meaning:
create_before_destroy → zero downtime
prevent_destroy → protects resources
ignore_changes → ignores manual updates
