# Secure Foundation - Terraform-First AWS Security Infrastructure
I've been talking to current Cloud Security Engineers to get a better idea of what companies are actually looking for. Most of them say the same thing: **learn to build infrastructure and become so good at it that you incorporate security into it.**

I want to be excellent in my job as a Cloud Security Engineer. In order to do so, I need to understand secure infrastructure at a foundational level. That's the motivation behind this project. I want to show that I'm willing to put in the work to learn the hard skills - Terraform, tflint, Golang, automated testing - and go a layer deeper every time. Because that's what differentiates average engineers from excellent ones. 

## What This Builds

A production-grade AWS foundation provisioned entirely with Terraform. No console clicking. No one-off scripts. Every resource defined, versioned, tested, and documented.

- VPC with public and private subnets across 2 AZs
- NAT Gateway, Internet Gateway, route tables
- Security groups with least-privilege rules
- IAM roles and instance profiles
- S3 bucket for Terraform remote state with DynamoDB locking
- CloudTrail enabled and logging to S3
- GuardDuty enabled
- AWS Config with managed rules (CIS benchmark subset)
- All resources tagged consistently

**Tools:** Terraform, tflint, terraform-docs, Terratest (Go), AWS (VPC, IAM, S3, CloudTrail, GuardDuty, Config), Linux (WSL2), Git

---
## Prerequisites
- AWS account with IAM user/role that has admin access
- Terraform >= 1.5.0
- AWS CLI configured with credentials
- tflint with the AWS ruleset
- terraform-docs
- Go (for Terratest)
- WSL2 or Linux environment

## How to Deploy

1. Clone the repo
2. Copy `backend.tf.example` to `backend.tf` and fill in your own S3 bucket and DynamoDB table names
3. `terraform init`
4. `terraform plan`
5. `terraform apply`

> **Note:** You need to manually create an S3 bucket (versioning + encryption enabled) and a DynamoDB table (partition key `LockID`, type String) before running `terraform init`. This is a one-time bootstrap - Terraform cannot create its own backend.

## Build Log

### Phase 1: Terraform Fundamentals + Environment

Started by installing Terraform, AWS CLI, tflint, and terraform-docs in WSL2 on Windows 11. I use WSL2 because learning Linux is non-negotiable if you want to be excellent in cloud. It acts and feels like an Ubuntu machine without spinning up a whole VM. More info [here](https://learn.microsoft.com/en-us/windows/wsl/install).

Spent time reading the official Terraform docs - providers, resources, variables, outputs, locals, and data sources. First time reading HCL syntax, but honestly it was super readable compared to something like Python. I was able to read and understand configs without constantly looking things up.

**S3 Backend + DynamoDB Lock Table (Manual Bootstrap)**

Configured an S3 bucket and DynamoDB lock table manually in my AWS account before writing any Terraform. The purpose: avoid a chicken-and-egg problem. Terraform can push or destroy hundreds of resources with one command, so creating the state backend manually up front prevents my automation from accidentally destroying core infrastructure. Think of it like network segmentation via VLANs - having a management VLAN prevents accidental mistakes.

**Preventing Data Disclosure**

Because this repo is public and going on my resume, I had my first opportunity to practice secure habits. Terraform state files (`*.tfstate`, `*.tfstate.*`) contain every resource ID, ARN, IP address, and sometimes plaintext outputs for everything Terraform manages. An attacker could use that to map my entire infrastructure. So I added state files and other semi-sensitive configs to `.gitignore` before the first commit. My local instance has all the files Terraform needs, but they never sync to the public repo.

**VPC + Subnets**

Created a VPC with two subnets - `10.0.1.0/24` for public and `10.0.2.0/24` for private. Tagged everything with project name, environment, and `ManagedBy = terraform`. Then ran `terraform init`, `terraform plan`, `terraform apply`, and `terraform destroy` repeatedly to understand the full lifecycle and build confidence that state was tracking correctly in S3.

## Security Decisions

*Documented as I go - every decision has a reason.*
| Decision | Why |
| --- | --- |
| S3 backend with DynamoDB locking | Prevents state corruption from concurrent runs and enables team collaboration |
| State files in .gitignore | State can contain plaintext secrets and full infrastructure mapping |
| All resources tagged | Consistent tagging enables cost tracking, ownership, and automated policy enforcement |
| Terragrunt for multi-account management | |
| Service Control Policies (SCPs) | |
| AWS Security Hub integration | | 
| CI/CD pipeline for ```terraform plan``` on PRs | | 
| *More to come as the project progresses...* |  |


The next thing I did was create a VPC with 2 subnets (private and public). You can find the config [here](main.tf). I made a super basic VPC, attached it to the "dev" environment and associated the "Terraform" tag with it. And for my subnets, I 10.0.1.0/24 for public and 10.0.2.0/24 for private. Once everything looked good, it was time to use Terraform to automate infrastructure set up and tear down.

I used the commands ```terraform init```, ```terraform plan```, ```terraform apply```, and ```terraform destroy``` repeatedly to understand what was happening and why.
