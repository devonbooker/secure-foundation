# Secure Foundation - Terraform-First AWS Security Infrastructure
I've been talking to current Cloud Security Engineers to get a better idea of what companies actually are looking for. Most of them say the same thing: **learn to build infrastructure and become so good at it that you incorporate security into it.** 

In this current economy, cloud and AI are super hot. Every ambitious company out there seems to be making advancements in both of these fields. And many people (like me) want to work in this ever-changing field. But the reality is, understanding the foundation needed for a good cloud and AI infrastructure is super important.

**I want to be excellent in my job as a Cloud Security Engineer.** In order to do so, I need to understand the foundations (secure infrsastructure) well. That's my motivation behind this project. I want to be able to show that I'm willing to put in the work to learn the hard skills like Terraform, TFlint, and Golang. I'm willing to go a layer deeper every time. Because this is what differentiates average engineers from excellent engineers. 

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

---
## Terraform Fundamentals + Environment
I started by installing Terraform, AWS CLI, TFlint, and terraform-docs into my WSL2. I use a Windows 11 machine. But I know how important learning Linux is if I'm going to be excellent in cloud. So I downloaded something called WSL2. It's basically a Linux plugin that runs on top of your PowerShell instance. It acts and feels like an Ubuntu machine, without having to spin up a whole VM. Super useful. More info can be found [here](https://learn.microsoft.com/en-us/windows/wsl/install). Installing the essential tools mentioned was super easy. I just googled "install terraform on WSL2" and it gave me a command I can just run in my terminal. I did that for all 4 tools. 

Then I spent some time reading the official Terraform docs. This was my first time reading HCL syntax. But honestly it was super readible compared to even something like Python. I've been able to read and understand the code without having to look it up. I'm glad they made Terraform, an infrastructure as code tool, easy to read because it only makes your job so much faster. I read a little bit about providers, resources, variables, outputs, locals, and data sources.

The next thing I did was configure an S3 bucket and DynamoDB lock table manually in my AWS account. The purpose of this is not to have a chicken + egg situation. Terraform is super powerful. You can push or destroy hundreds of lines of code with one command. So creating this bucket and lock table combo in the beginning of my project prevents important core infrastructure configurations to be unaffected by my automation. Think of it like network segmentation via VLANs - having a management VLAN prevents accidential mistakes. 

Now because this project is meant to be public, and it's going on my resume, I had my first opportunity to prevent a data disclosure. In Terraform there are 2 config files called "\*.tfstate" and "\*.tfstate.*". This file contains every resource IP, ARN, IP address and sometimes plaintext outputs for everything Terraform manages. This could allow attacks to see how my infrastructure is mapped and how it's connected. No bueno. So I added it to my .gitignore folder found [here](.gitignore). I also included some other config files in the ignore list for semi-sensitive content that doesn't really need to be advertised on GitHub. Adding something to .gitignore allows it to be ignored when you run the command "git push" in WSL2. My local instance of my repo will have all the required files for Terraform, but it won't be synced with my public GitHub repo.

The next thing I did was create a VPC with 2 subnets (private and public). You can find the config [here](main.tf). 
