Final Folder Structure
terraform-modules/
│── main.tf
│── variables.tf
│── outputs.tf
│── providers.tf
│
└── modules/
    ├── ec2-instance/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── security-group/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
 2. PROVIDERS (root/providers.tf)
provider "aws" {
  region = "ap-south-1"
}
 3. SECURITY GROUP MODULE
modules/security-group/variables.tf
variable "vpc_id" {}
variable "sg_name" {}
variable "ingress_ports" {
  type    = list(number)
  default = [22, 80]
}
variable "tags" {
  type    = map(string)
  default = {}
}
modules/security-group/main.tf
resource "aws_security_group" "this" {
  name   = var.sg_name
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
modules/security-group/outputs.tf
output "sg_id" {
  value = aws_security_group.this.id
}
 4. EC2 MODULE
modules/ec2-instance/variables.tf
variable "ami_id" {}
variable "instance_type" {
  default = "t2.micro"
}
variable "subnet_id" {}
variable "security_group_ids" {
  type = list(string)
}
variable "instance_name" {}
variable "tags" {
  type    = map(string)
  default = {}
}
modules/ec2-instance/main.tf
resource "aws_instance" "this" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = merge(var.tags, {
    Name = var.instance_name
  })
}
modules/ec2-instance/outputs.tf
output "instance_id" {
  value = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}
 5. ROOT MAIN FILE
main.tf
data "aws_ami" "amazon_linux" {
  most_recent = true

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }

  owners = ["amazon"]
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

module "web_sg" {
  source        = "./modules/security-group"
  vpc_id        = aws_vpc.main.id
  sg_name       = "terraweek-web-sg"
  ingress_ports = [22, 80, 443]
  tags          = { Project = "terraweek" }
}

module "web_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "terraweek-web"
  tags               = { Project = "terraweek" }
}

module "api_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "terraweek-api"
  tags               = { Project = "terraweek" }
}
 6. OUTPUTS (root/outputs.tf)
output "web_ip" {
  value = module.web_server.public_ip
}

output "api_ip" {
  value = module.api_server.public_ip
}
 7. RUN COMMANDS
terraform init
terraform plan
terraform apply