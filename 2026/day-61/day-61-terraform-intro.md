🔹 STEP 1: Fix Installation
1. Install Terraform (Windows)

 Open PowerShell as Administrator

choco install terraform -y
2. Verify installation

Close terminal → reopen Git Bash / CMD:

terraform -version

Expected output:

Terraform v1.x.x
3. Install AWS CLI (if not installed)

Check:

aws --version

If not installed → install AWS CLI and then continue.

4. Configure AWS
aws configure

Enter:

AWS Access Key ID: XXXXX
AWS Secret Access Key: XXXXX
Default region: ap-south-1
Output format: json
5. Verify AWS
aws sts get-caller-identity

✔ If you see account info → setup is correct

🔹 STEP 2: Create Terraform Project
mkdir terraform-basics
cd terraform-basics
🔹 STEP 3: Create main.tf

Create file:

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

# ---------------- S3 Bucket ----------------
resource "aws_s3_bucket" "demo_bucket" {
  bucket = "megha-terraform-bucket-2026-12345"
}

# ---------------- EC2 Instance ----------------
resource "aws_instance" "demo_ec2" {
  ami           = "ami-0f5ee92e2d63afc18"
  instance_type = "t2.micro"

  tags = {
    Name = "TerraWeek-Day1"
  }
}

Change bucket name (must be unique globally)

🔹 STEP 4: Terraform Commands (MAIN FLOW)
1. Initialize Terraform
terraform init

 Downloads AWS provider plugins

2. Check execution plan
terraform plan

 Shows what will be created

3. Apply configuration
terraform apply

Type:

yes
4. Verify in AWS Console
Go to S3 → bucket created
Go to EC2 → instance running

Take these:


terraform init

terraform apply

AWS Console:

S3 bucket visible
EC2 instance running

aws sts get-caller-identity
 STEP 5: Understand State File

Run:

terraform state list
terraform show
What state file contains:
EC2 instance ID
S3 bucket details
AMI ID
region
tags
Why important:

Terraform uses it to track what it created.


 STEP 6: Modify Infrastructure

Edit main.tf:

tags = {
  Name = "TerraWeek-Modified"
}

Then run:

terraform plan
terraform apply
 STEP 7: Destroy Infrastructure
terraform destroy

Type:

yes

 This deletes:

EC2 instance
S3 bucket
 STEP 8: Create Documentation File

Create:

day-61-terraform-intro.md
Paste this content:
 What is IaC?

Infrastructure as Code means managing cloud resources using code instead of manual clicks.

Terraform commands:
init → downloads plugins
plan → shows changes
apply → creates resources
destroy → deletes resources
state list → shows tracked resources
 Why Terraform state is important:

It tracks real-world infrastructure and keeps Terraform in sync.
