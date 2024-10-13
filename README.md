# terraform-aws-project
This is a project which creates the infrastructure on AWS using Terraform

Terraform Project: Deploying a VPC and Application Components on AWS
In this project, we will create a Terraform configuration to deploy an AWS infrastructure that includes a VPC, private and public subnets, an Internet Gateway, route tables, S3 buckets, EC2 instances, and an Elastic Load Balancer (ELB).

Prerequisites
Terraform installed on your machine.
AWS CLI configured with your AWS account.
Step 1: Create the AWS Provider Block
First, let’s create the AWS provider block. Search for "Terraform AWS provider" on Google and navigate to the Terraform documentation: Terraform AWS Provider. Copy the following code and paste it into a file named provider.tf.

hcl

provider "aws" {
  # Configuration options
  region = "ap-south-1"
}
Step 2: Initialize Terraform
Run the following command to prepare your working directory for other Terraform commands:


terraform init
Step 3: Create the VPC and Subnets
Next, we’ll define the VPC and subnets in the main.tf file. Again, refer to the Terraform documentation for the relevant resources.

VPC

resource "aws_vpc" "myvpc" {
  cidr_block = var.cidr
}
Subnets

resource "aws_subnet" "sub1" {
  vpc_id     = aws_vpc.myvpc.id
  cidr_block = var.sub_cidr_1
  availability_zone = "ap-south-1"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "sub2" {
  vpc_id     = aws_vpc.myvpc.id
  cidr_block = var.sub_cidr_2
  availability_zone = "ap-south-1"
  map_public_ip_on_launch = true
}
To allow access to our EC2 instances, we need to create an Internet Gateway and route tables.

Step 4: Create the Internet Gateway
hcl
Copy code
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.myvpc.id
}
Step 5: Create the Route Table

resource "aws_route_table" "RT" {
  vpc_id = aws_vpc.myvpc.id

  route {
    cidr_block = "0.0.0.0/0"  # Allow all incoming traffic
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "rta1" {
  subnet_id      = aws_subnet.sub1.id
  route_table_id = aws_route_table.RT.id
}

resource "aws_route_table_association" "rta2" {
  subnet_id      = aws_subnet.sub2.id
  route_table_id = aws_route_table.RT.id
}
Now, run terraform validate to check for errors, followed by terraform plan to see what will be created, and then execute:


terraform apply
Step 6: Define a Security Group for EC2 Instances

resource "aws_security_group" "websg" {
  name   = "websg"
  vpc_id = aws_vpc.myvpc.id

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
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
    Name = "websg"
  }
}
Step 7: Create EC2 Instances

resource "aws_instance" "webserver1" {
  ami                    = "ami-0dee22c13ea7a9a67"
  instance_type         = "t2.micro"
  vpc_security_group_ids = [aws_security_group.websg.id]
  subnet_id             = aws_subnet.sub1.id
  user_data             = base64encode(file("userdata.sh"))
}

resource "aws_instance" "webserver2" {
  ami                    = "ami-0dee22c13ea7a9a67"
  instance_type         = "t2.micro"
  vpc_security_group_ids = [aws_security_group.websg.id]
  subnet_id             = aws_subnet.sub2.id
  user_data             = base64encode(file("userdata2.sh"))
}
Step 8: Create an S3 Bucket

resource "aws_s3_bucket" "example" {
  bucket = "ali-badshah-bucket"
}
Run terraform plan and then:

terraform apply
Step 9: Create an Application Load Balancer

resource "aws_lb" "myalb" {
  name               = "myalb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.websg.id]
  subnets            = [aws_subnet.sub1.id, aws_subnet.sub2.id]

  tags = {
    Environment = "web"
  }
}
Step 10: Create a Target Group

resource "aws_lb_target_group" "tg" {
  name     = "myTG"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.myvpc.id

  health_check {
    path = "/"
    port = "traffic-port"
  }
}
Next, we’ll attach our EC2 instances to the target group.

resource "aws_lb_target_group_attachment" "attach1" {
  target_group_arn = aws_lb_target_group.tg.arn
  port             = 80
  target_id        = aws_instance.webserver1.id
}

resource "aws_lb_target_group_attachment" "attach2" {
  target_group_arn = aws_lb_target_group.tg.arn
  port             = 80
  target_id        = aws_instance.webserver2.id
}
Step 11: Create a Load Balancer Listener

resource "aws_lb_listener" "listener" {
  load_balancer_arn = aws_lb.myalb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    target_group_arn = aws_lb_target_group.tg.arn
    type             = "forward"
  }
}
Finally, output the Load Balancer DNS name:

output "loadbalancerdns" {
  value = aws_lb.myalb.dns_name
}
Run terraform validate, terraform plan, and then:


terraform apply
Cleanup
After completing the demo, you can delete all resources using:

terraform destroy
