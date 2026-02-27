# wordpress-tf

Project description

This repository contains Terraform configuration and supporting documentation to deploy a production-ready WordPress site on AWS. The deployment focuses on secure networking (VPC, public and private subnets), an EC2 instance (or autoscaling behind an ALB), optional managed database (RDS), object storage for media (S3), and a CDN (CloudFront) to accelerate and secure content delivery.

Architecture overview

- VPC with public and private subnets across multiple AZs
- Internet Gateway (IGW) for public subnet access
- NAT Gateway (or NAT instances) for outbound internet access from private subnets
- Route Tables for correct routing between subnets and gateways
- Security Groups restricting access (HTTP/HTTPS from internet to ALB/EC2, SSH from admin IP only, DB access only from app subnet)
- EC2 instance(s) running WordPress (user-data bootstrap or AMI)
- Optional RDS (MySQL/MariaDB) in private subnets for the WordPress database
- S3 bucket for WordPress uploads and backups
- CloudFront distribution in front of the site (origin can be ALB/EC2 or S3) with ACM TLS certificate for HTTPS
- Route53 hosted zone to point a custom domain to CloudFront or ALB

Primary uses

- Deploy a secure, highly-available WordPress site on AWS using Terraform
- Demonstrates best practices for network segmentation, least-privilege security groups, and CDN fronting
- Supports separation of compute and database for easier scaling and maintenance

Prerequisites

- An AWS account with permissions to create VPCs, EC2, RDS, S3, CloudFront, ACM, IAM, and Route53 resources
- Terraform (>= 1.0) installed locally
- AWS CLI configured with credentials (or use an assumed role/CI pipeline credentials)
- (Optional) Domain registered and managed in Route53 if you want automated DNS and ACM validation

Deployment (high level)

1. Clone the repo:
   git clone <repo-url>
   cd wordpress-tf

2. Review variables (vars.tf / terraform.tfvars) and set values for:
   - aws_region
   - vpc_cidr, public/private subnet cidrs
   - instance_type, key_name (for SSH)
   - db_engine, db_size, db_username (if using RDS)
   - domain_name and hosted_zone_id (if using Route53)

3. Initialize and plan:
   terraform init
   terraform plan -var-file=terraform.tfvars

4. Apply:
   terraform apply -var-file=terraform.tfvars

5. After apply:
   - Note outputs: public IP / ALB DNS, CloudFront domain, RDS endpoint
   - If using CloudFront + ACM, wait for certificate validation and distribution deployment
   - SSH to EC2 (if applicable) to confirm WordPress is running or use the ALB/CloudFront endpoint in a browser

WordPress setup notes

- Use a bootstrapping script (user-data) to install PHP, Nginx/Apache, and WordPress, or bake an AMI
- Point WordPress uploads to S3 (recommended) using a plugin and restrict direct S3 public access
- Store database credentials securely via AWS Secrets Manager or SSM Parameter Store and reference them in user-data or config

CDN (CloudFront) configuration notes

- Create CloudFront distribution with origin set to the ALB (preferred) or EC2 public endpoint / S3 bucket
- Use an ACM certificate (in us-east-1 for CloudFront) and attach it to the distribution for HTTPS
- Configure caching behaviors and invalidation as needed for dynamic WordPress content

Security and best practices

- Restrict SSH (port 22) to a known admin IP only
- Serve the site only over HTTPS (redirect HTTP to HTTPS at ALB/CloudFront)
- Place databases in private subnets with no public access
- Use IAM roles for EC2 to grant least-privilege access to S3 and Secrets Manager
- Regularly snapshot and backup RDS and S3

Cleanup

- To destroy all resources created by Terraform:
  terraform destroy -var-file=terraform.tfvars

Customization

- Add autoscaling group + ALB for high availability
- Replace EC2-installed MySQL with RDS for managed database
- Use ECS/Fargate or EKS for containerized WordPress deployments

Contributing

- Open issues or PRs for improvements. Ensure Terraform code follows formatting (terraform fmt) and validate (terraform validate) before submitting.

License

- This repo does not include a license by default. Add a LICENSE file if you want to share the code.
