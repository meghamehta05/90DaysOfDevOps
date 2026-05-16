STEP 1: Initialize your project
cd ~/Documents/2026/day-64
terraform init
STEP 2: Create backend (S3 + DynamoDB)
S3 Bucket
aws s3api create-bucket \
  --bucket terraweek-state-megha-2026 \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1
Enable versioning
aws s3api put-bucket-versioning \
  --bucket terraweek-state-megha-2026 \
  --versioning-configuration Status=Enabled
DynamoDB lock table
aws dynamodb create-table \
  --table-name terraweek-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
 STEP 3: Add backend in Terraform
terraform {
  backend "s3" {
    bucket         = "terraweek-state-megha-2026"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraweek-state-lock"
    encrypt        = true
  }
}
 STEP 4: Migrate state
terraform init -migrate-state

 Type: yes

STEP 5: Verify state migration
terraform state list

Check AWS:

S3 bucket → should contain terraform.tfstate
STEP 6: Test locking (IMPORTANT SS)

Terminal 1:

terraform apply

Terminal 2 (while 1 is running):

terraform plan


 STEP 7: Import existing resource
Create bucket manually in AWS:
terraweek-import-test-megha
Add empty resource:
resource "aws_s3_bucket" "imported" {
  bucket = "terraweek-import-test-megha"
}
Import:
terraform import aws_s3_bucket.imported terraweek-import-test-megha
 STEP 8: State drift test

Change in AWS console:

EC2 tag → “ManuallyChanged”

Then:

terraform plan

Fix:

terraform apply
STEP 9: PUSH TO GITHUB
git add .
git commit -m "Day 64 - Terraform remote state, locking, import and drift management completed"
git push origin main

(or master)

git push origin master