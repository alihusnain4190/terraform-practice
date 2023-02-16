# variable
how to declare variable in terraform
```
variable "aws_region" {
  description = "default of region is virginia "
  type        = string

  default = "us-east-1"
}
```

# create Security grouo
```
resource "aws_security_group" "ssh" {
  name        = var.security_group
  description = "this securoty group use for port 22"
  #vpc_id      = aws_vpc.main.id  ## if you do not define VPC it will create SG in default VPC
  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    description = "port 22"
    from_port   = 22
    protocol    = "tcp"
    to_port     = 22
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
  tags = {
    name = var.security_group
  }
}
resource "aws_security_group" "web" {
  name        = var.web_security_group
  description = "this securoty group use for port 80"
  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    description = "port 80"
    from_port   = 80
    protocol    = "tcp"
    to_port     = 80
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
  tags = {
    name = var.web_security_group
  }
}

```

# Ec2 instance
```
resource "aws_instance" "web" {
  ami                    = data.aws_ami.my_ami.id
  instance_type          = var.instance_type
  key_name               = "bash-test"
  vpc_security_group_ids = [aws_security_group.ssh.id, aws_security_group.web.id]

  tags = {
    Name = var.instance_name
  }
}
```
# Data source
1) Data source allow data to be fetched or compute for use elsewhere in terraform configration.

2) We use data source to fetch data from AWS .

3) We also use data source to get data  from project 1 to project2 

4) Data source is also support meta argument like  count or for_each.![image](https://user-images.githubusercontent.com/59732833/219507062-c8aaa340-533e-4457-9ddc-db8f5273c2c2.png)

```
data "aws_ami" "my_ami" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-kernel-5.10-hvm-*-gp2"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}
```
# output varibale

```
# define out put of variable

output "public_IP" {
  description = "public IP of Ec2 instance"
  value       = aws_instance.web.public_ip
}

output "public_dns" {
  description = "public DNS of Ec2 instnace"
  value       = aws_instance.web.public_dns
}
