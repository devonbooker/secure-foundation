# Secure Foundation - Terraform-First AWS Security Infrastructure
I've been talking to current Cloud Security Engineers to get a better idea of what companies actually are looking for. Most of them say the same thing: learn to build infrastructure and become so good at it that you incorporate security into it. In this current economy, cloud and AI are super hot. Every ambitious company out there seems to be making advancements in both of these fields. And many people like me want to work in this ever-changing field. But the reality is, understanding the foundation needed for a good cloud and AI infrastructure is super important.

I want to be excellent in my job as a Cloud Security Engineer. In order to do so, I need to understand the foundations (secure infrsastructure) well. That's my motivation behind this project. I want to be able to show that I'm willing to put in the work to learn the hard skills like Terraform, TFlint, and Golang. I'm willing to go a layer deeper every time. Because this is what differentiates average engineers from excellent engineers. 

## Project Overview
- VPC with public and private subnets across 2 AZs
- NAT Gateway, Internet Gateway, route tables
- Security groups with least-privilege rules
- IAM roles and instance profiles
- S3 bucket for Terraform remote state with DynamoDB locking
- CloudTrail enabled and logging to S3
- GuardDuty enabled
- AWS Config with managed rules (CIS benchmark subset)
- All resources tagged consistently

**Tools:** Terraform, tflint, terraform-docs, Terratest (Go), AWS (VPC, IAM, S3, CloudTrail, GuardDuty, Config), Linux, Git
