# Terraform AWS Project 

![Terraform](https://img.shields.io/badge/Terraform-IaC-blue?logo=terraform)
![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws)
![License](https://img.shields.io/badge/License-MIT-green)

This project provisions a complete AWS environment using **Terraform**, including:
- A custom **VPC** with public subnet
- An **Internet Gateway** and route table association
- An **EC2 instance** with a public IP
- A managed **Security Group** allowing SSH from your IP
- An **S3 bucket** for logs

---

## Project Structure
terraform-aws-project/ ├── main.tf # Root configuration ├── variables.tf # Root variables ├── outputs.tf # Root outputs ├── modules/ │ ├── vpc/ # VPC + subnet + IGW + route table │ │ ├── main.tf │ │ ├── variables.tf │ │ └── outputs.tf │ ├── ec2/ # EC2 instance │ │ ├── main.tf │ │ ├── variables.tf │ │ └── outputs.tf │ ├── security_group/ # Security group for SSH/HTTP │ │ ├── main.tf │ │ ├── variables.tf │ │ └── outputs.tf │ └── s3/ # S3 bucket

---

## Prerequisites
- AWS account with IAM user credentials
- Terraform installed (`>= 1.5.0`)
- A valid AWS key pair (`.pem` file) for SSH access

---

## Usage
1. **Initialize Terraform**
   ```bash
   terraform init
   
Plan the infrastructure
```bash
terraform plan
```

Apply changes
``bash
terraform apply
```

Outputs After apply, Terraform will display:

```
vpc_id → VPC ID

public_subnet_id → Subnet ID

ec2_public_ip → Public IP of EC2 instance

s3_bucket_name → S3 bucket name
```

Connecting to EC2
```bash
ssh -i ~/Desktop/server-key.pem ec2-user@<ec2_public_ip>
Terraform Snippets
VPC Module (modules/vpc/main.tf)
hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.project_name}-vpc"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.project_name}-igw"
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.project_name}-public-subnet"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "${var.project_name}-public-rt"
  }
}

resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
Security Group Module (modules/security_group/main.tf)
hcl
resource "aws_security_group" "ec2_sg" {
  name        = "${var.project_name}-ec2-sg"
  description = "Allow SSH and HTTP access"
  vpc_id      = var.vpc_id

  ingress {
    description = "SSH from my IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip]
  }

  ingress {
    description = "HTTP from anywhere"
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
    Name = "${var.project_name}-ec2-sg"
  }
}
EC2 Module (modules/ec2/main.tf)
hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 AMI (example)
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  key_name      = var.key_name
  vpc_security_group_ids = [var.sg_id]

  tags = {
    Name = "${var.project_name}-ec2"
  }
}

output "public_ip" {
  value = aws_instance.web.public_ip
}
S3 Module (modules/s3/main.tf)
hcl
resource "aws_s3_bucket" "logs" {
  bucket = "${var.project_name}-terraform-logs-${random_id.bucket_suffix.hex}"

  tags = {
    Name = "${var.project_name}-s3-logs"
  }
}

resource "random_id" "bucket_suffix" {
  byte_length = 4
}

output "bucket_name" {
  value = aws_s3_bucket.logs.bucket
}

```

Why This Project Matters
Demonstrates modular Terraform design (VPC, EC2, SG, S3).

Shows clean outputs and variable management.

Proves ability to provision real AWS infrastructure.

Great portfolio piece for DevOps / Cloud engineering roles.

Future Improvements
Add private subnets and NAT Gateway for backend services

Provision an RDS database in private subnets

Add an Application Load Balancer (ALB) for EC2 scaling

Integrate with Terraform Cloud or remote state backend

Add CI/CD pipeline for automated deployments

License
This project is licensed under the MIT License.
