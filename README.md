# terraform-aws-project
This is a project which creates the infrastructure on AWS using Terraform

Terraform Project: Deploying a VPC and Application Components on AWS
In this project, we will create a Terraform configuration to deploy an AWS infrastructure that includes a VPC, private and public subnets, an Internet Gateway, route tables, S3 buckets, EC2 instances, and an Elastic Load Balancer (ELB).

Prerequisites
Terraform installed on your machine.
AWS CLI configured with your AWS account.


Step 1: Create the AWS Provider Block
First, let’s create the AWS provider block. Search for "Terraform AWS provider" on Google and navigate to the Terraform documentation: Terraform AWS Provider. Copy the following code and paste it into a file named provider.tf.

Step 2: Initialize Terraform
Run the following command to prepare your working directory for other Terraform commands:

terraform init

Step 3: Create the VPC and Subnets
Next, we’ll define the VPC and subnets in the main.tf file. Again, refer to the Terraform documentation for the relevant resources.

To allow access to our EC2 instances, we need to create an Internet Gateway and route tables.

Step 4: Create the Internet Gateway

Step 5: Create the Route Table

Now, run terraform validate to check for errors, followed by terraform plan to see what will be created, and then execute:
terraform apply

Step 6: Define a Security Group for EC2 Instances

Step 7: Create EC2 Instances

Step 8: Create an S3 Bucket

Run terraform plan and then terraform apply

Step 9: Create an Application Load Balancer

Step 10: Create a Target Group

Next, we’ll attach our EC2 instances to the target group.

Step 11: Create a Load Balancer Listener

Run terraform validate, terraform plan, and then:


terraform apply
Cleanup
After completing the demo, you can delete all resources using:

terraform destroy
