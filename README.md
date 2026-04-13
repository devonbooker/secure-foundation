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

### Phase 2: Modules, Validation, and Tooling

**VPC Module Refactor**

Restructured the flat VPC config into a reusable module at `modules/vpc/`. Module has its own `main.tf`, `variables.tf`, `outputs.tf`, and `versions.tf`. Updated the root to call the module and pass all inputs through. Since the VPC had already been applied, used `terraform state mv` to move the three existing resources to their new module addresses (`module.vpc.aws_vpc.main`, etc.) before applying. Ran `terraform plan` after the move to confirm zero infrastructure changes - the refactor was purely structural.

**NAT Gateway, IGW, Route Tables, Security Groups**

Expanded the VPC module to include a full networking stack. Internet Gateway attached to the VPC for public subnet outbound access. NAT Gateway (with Elastic IP) deployed into the public subnet so private subnet resources can reach the internet without being directly reachable. Separate route tables for public (`0.0.0.0/0 → IGW`) and private (`0.0.0.0/0 → NAT GW`) subnets. Two security groups: public allows inbound 80/443 only, private allows inbound from within the VPC CIDR only. Both allow all outbound.

**Variable Validation Blocks**

Added validation blocks across root and module variables. CIDR variables use `can(cidrnetmask())` to reject malformed input before Terraform ever calls the AWS API. Environment is enforced as an enum (`dev`, `staging`, `prod`) to prevent typos from silently creating mistaggged resources. Project name is validated against a regex to keep it safe for use in AWS resource names and DNS. Added a required `tags` variable of type `map(string)` with validation requiring `Environment` and `Owner` keys - enforces organizational tagging policy at the variable layer rather than relying on convention.

**TFLint + Pre-commit Hook**

Configured TFLint with the AWS ruleset (`tflint-ruleset-aws v0.40.0`) via `.tflint.hcl`. Set up a pre-commit hook that runs `tflint --recursive` before every commit and blocks if any issues are found. Added terraform-docs to the same hook so `modules/vpc/README.md` is always regenerated and staged automatically. Zero tolerance policy: nothing commits with lint output.

## Security Decisions

*Documented as I go - every decision has a reason.*
| Decision | Why |
| --- | --- |
| S3 backend with DynamoDB locking | Prevents state corruption from concurrent runs and enables team collaboration |
| State files in .gitignore | State can contain plaintext secrets and full infrastructure mapping |
| All resources tagged | Consistent tagging enables cost tracking, ownership, and automated policy enforcement |
| Required `tags` variable with validation | Enforces `Environment` and `Owner` keys on every resource at plan time - policy can't be skipped or forgotten |
| Public SG: 80/443 inbound only, no SSH | Least privilege - SSH added per use case scoped to a specific IP, not left open by default |
| Private SG: inbound from VPC CIDR only | Private resources are unreachable from the internet regardless of what gets deployed into the private subnet |
| NAT Gateway in public subnet, not private | Private subnet gets outbound internet access without any inbound exposure - clean one-way traffic flow |
| Variable validation on CIDR blocks and environment | Catches malformed input at plan time rather than letting a bad value reach the AWS API and fail obscurely |
| TFLint pre-commit hook with zero tolerance | Lint issues can't be committed - enforcement is automatic, not reliant on developer discipline |
| `terraform state mv` before module refactor | Prevents destroy/recreate of live infrastructure when restructuring code - state addresses must match resource addresses |
| Terragrunt for multi-account management | |
| Service Control Policies (SCPs) | |
| AWS Security Hub integration | | 
| CI/CD pipeline for ```terraform plan``` on PRs | | 
| *More to come as the project progresses...* |  |

