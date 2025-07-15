Here is your updated documentation in proper **Markdown (`.md`) format**:

---

````md
# ✅ Terraform-EC2-instance-with-Keypair-and-User-Data

This project provisions an EC2 instance using Terraform with the following features:

- Instance Type: `t2.micro`
- Automatically generated SSH key pair (private key downloadable)
- Security Group allowing inbound HTTP (port 80)
- User data script to install Apache web server
- Validation, planning, and cleanup steps included
- Reflections and challenges documented

---

## ✅ Step 1: Verify AWS CLI Installation and Authentication

```bash
aws --version
aws-cli/2.26.2 Python/3.13.2 Windows/11 exe/AMD64
````

**Confirm identity:**

```bash
aws sts get-caller-identity
```

📸 Screenshot:
![aws-cli-auth-sts](https://github.com/user-attachments/assets/aws-cli-auth-sts.png)

---

## ✅ Step 2: Project Setup

**Create project directory:**

```bash
mkdir terraform-ec2-instance
cd terraform-ec2-instance
```

📸 Screenshot:
![create-dir](https://github.com/user-attachments/assets/060e4f9a-b0ef-4ac2-a1e7-e1b82b49e51f)

---

## ✅ Step 3: Complete Terraform Code (in `main.tf`)

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "ec2_key" {
  key_name   = "ec2-key"
  public_key = tls_private_key.ec2_key.public_key_openssh
}

resource "aws_security_group" "allow_http" {
  name        = "allow-http"
  description = "Allow inbound HTTP"

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
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  key_name      = aws_key_pair.ec2_key.key_name
  security_groups = [aws_security_group.allow_http.name]

  user_data = <<-EOF
              #!/bin/bash
              sudo yum update -y
              sudo yum install -y httpd
              sudo systemctl start httpd
              sudo systemctl enable httpd
              echo "<h1>Apache is running from Terraform EC2</h1>" > /var/www/html/index.html
              EOF

  tags = {
    Name = "WebServerWithApache"
  }
}

output "instance_public_ip" {
  value = aws_instance.web.public_ip
}

output "private_key_pem" {
  value     = tls_private_key.ec2_key.private_key_pem
  sensitive = true
}
```

📸 Screenshot:
![terraform-code](https://github.com/user-attachments/assets/a84d5b70-06a3-4353-81b0-8716a6c9a266)

---

## ✅ Step 4: Terraform Commands

### Initialize

```bash
terraform init
```

📸 Screenshot:
![terraform-init](https://github.com/user-attachments/assets/a47e7b24-0806-4dc7-a8d9-5a26f99578c6)

---

### Validate

```bash
terraform validate
```

📸 Screenshot:
![terraform-validate](https://github.com/user-attachments/assets/terraform-validate.png)

---

### Plan

```bash
terraform plan
```

📸 Screenshot:
![terraform-plan](https://github.com/user-attachments/assets/terraform-plan.png)

---

### Apply

```bash
terraform apply
```

📸 Screenshot:
![terraform-apply](https://github.com/user-attachments/assets/5f8458ec-2d65-4f4c-82a4-a3b918d0bb79)

---

## ✅ Step 5: Web Server Verification

Visit the public IP output in your browser:

```
http://<instance_public_ip>
```

📸 Screenshot:
![apache-browser](https://github.com/user-attachments/assets/84b3ae1e-8106-492e-aeba-f4a43787f9c2)

---

## ✅ Step 6: Cleanup with Terraform Destroy

```bash
terraform destroy
```

📸 Screenshot:
![terraform-destroy](https://github.com/user-attachments/assets/terraform-destroy.png)

---
