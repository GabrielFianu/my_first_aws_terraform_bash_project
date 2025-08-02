# Provision Secure Ubuntu EC2 with Nginx Using Terraform + Bash

## Introduction

This project demonstrates how to:

a. Provision a secure EC2 instance (Ubuntu) in AWS
b. Use Terraform for Infrastructure as Code (IaC)
c. Automatically install and configure Nginx via a Bash user-data script
d. Display a custom welcome page upon visiting the EC2 public IP

📂 Project Structure

```
terraform-nginx-ubuntu/
│
├── main.tf            → Core infrastructure resources
├── variables.tf       → Input variables
├── outputs.tf         → Outputs like the public IP
├── user-data.sh       → Bash script to install and configure Nginx
└── README.md          → This documentation
```

**Prerequisites**
a. Terraform	
b. AWS CLI	
c. AWS Account	
d. SSH Key Pair	
e.Configure your AWS CLI with credentials:
```
aws configure
```

---

### 1. Configuration files
**Note** Create new files by clicking the icon below your current directory.

<img width="217" height="181" alt="Image" src="https://github.com/user-attachments/assets/fd6a5789-adcb-406f-b80d-955a783418af" />

a. Create **provider.tf** file and the following below: 

```
provider "aws" {
  region = var.region
}
```

<img width="702" height="181" alt="Image" src="https://github.com/user-attachments/assets/a7d0ede2-9b02-4b78-b8a0-8c9fa6e80067" />


b. Create **main.tf** file and the following below: 

```
resource "aws_instance" "nginx_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  user_data     = file("user-data.sh")

  tags = {
    Name = var.instance_name
  }

  vpc_security_group_ids = [aws_security_group.allow_http.id]
}


data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_security_group" "allow_http" {
  name        = "allow_http"
  description = "Allow HTTP inbound traffic"

  ingress {
    description = "Allow HTTP"
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
}
```

<img width="712" height="699" alt="Image" src="https://github.com/user-attachments/assets/062b9cf6-6786-4f8e-b056-9c1dcd0bdb31" />


c. Create and configure variables.tf file
```
variable "aws_region" {
  description = "The AWS region to deploy resources"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "instance_name" {
  description = "Name tag for EC2 instance"
  type        = string
  default     = "web-nginx"
}
```

<img width="714" height="462" alt="Image" src="https://github.com/user-attachments/assets/611fe56e-ac69-4585-8bb0-d3e9efc14fe7" />


d. Create and Configure Outputs.tf file
```
output "public_ip" {
  description = "Public IP of EC2 instance"
  value       = aws_instance.nginx_server.public_ip
}
```

<img width="521" height="263" alt="Image" src="https://github.com/user-attachments/assets/581fef40-ed3f-4b78-8cfc-4a3a94d70d45" />


Create and Configure user-data.sh file
```
#!/bin/bash
sudo apt update -y
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
echo "<h1>Welcome to your Terraform Nginx Server</h1>" | sudo tee /var/www/html/index.html
```

<img width="708" height="283" alt="Image" src="https://github.com/user-attachments/assets/323caeff-be30-44b6-a29b-88d74dc2f70d" />

**Note** before the next steps you have the following configuration files.

<img width="227" height="314" alt="Image" src="https://github.com/user-attachments/assets/a3efa354-f40f-4adc-8f14-52c83ef7321b" />


---

### 2. Initialize Terraform
Run

```
terraform init
```

<img width="649" height="699" alt="Image" src="https://github.com/user-attachments/assets/75297d57-05ff-4853-a2cc-d995fc9ac36d" />

**Note** after initializing, your files under your current repository should look like this.

<img width="225" height="310" alt="Image" src="https://github.com/user-attachments/assets/4a46be9c-1918-4cfc-8c18-62a7d94723a8" />


---

### 3. Validate & Plan

Run

```
terraform validate
```

<img width="463" height="94" alt="Image" src="https://github.com/user-attachments/assets/efb95e76-e5a7-48d3-b55d-821bf76c0c6c" />


Run

```
terraform plan
```

<img width="639" height="713" alt="Image" src="https://github.com/user-attachments/assets/24d098dc-3da2-4fb6-8796-69b836ee4db7" />

**Note**: you will be prompted for your keypair.


---

### 4. Deploy EC2 Instance


```
terraform apply
```
Type and enter **yes** when prompted.


<img width="655" height="697" alt="Image" src="https://github.com/user-attachments/assets/9e0a12a6-a714-4019-b41c-1b2af26b2bde" />

**Note**: you will be prompted for your keypair.

---


### 5. Visit the Public IP
Terraform will print something like:

Outputs:
public_ip = "IP address"

Copy the IP address. Open your browser, paste the IP address and press the Enter key:

You’ll see: 🚀 Hello from Terraform + Ubuntu + Nginx!

<img width="649" height="177" alt="Image" src="https://github.com/user-attachments/assets/1df1988a-4a07-4bb8-bb68-17f0f80e1448" />

---

###. 6. Confirm resources provisioned on terraform

Run

```
terraform state list
```

<img width="650" height="237" alt="Image" src="https://github.com/user-attachments/assets/f9bd519d-3118-4873-a54c-86ecb30b05c0" />

and


```
terraform show
```

<img width="651" height="707" alt="Image" src="https://github.com/user-attachments/assets/c2992eda-7070-4dae-89bd-49851a494c7a" />


---

### 7. Cleanup (Avoid Charges)
a. To destroy all provisioned infrastructure:


Run
```
terraform destroy
```

<img width="645" height="707" alt="Image" src="https://github.com/user-attachments/assets/c061c97b-b604-4512-b285-a5edaa011ce8" />

<img width="649" height="691" alt="Image" src="https://github.com/user-attachments/assets/74f02a74-8564-4e1b-a0b7-5b6ef92543f7" />


b. Run the following to confirm
bash
```
terraform state list
```
and
```
terraform show
```

<img width="655" height="238" alt="Image" src="https://github.com/user-attachments/assets/527fd4a9-db3a-475f-89ec-1133908f8fdc" />

---

###. 6. Confirm resources provisioned and destroyed from the Console

![Image](https://github.com/user-attachments/assets/50285673-6f69-46e1-997f-01df34dfe124)

![Image](https://github.com/user-attachments/assets/70ca9b74-b242-4649-8b3f-008a9bee6aee)

![Image](https://github.com/user-attachments/assets/db965afa-cfad-49fe-aa1c-a2f87257533f)

![Image](https://github.com/user-attachments/assets/f27bc53f-eaa9-45e9-a32b-6029070cc290)
