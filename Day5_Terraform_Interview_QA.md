# Terraform
---

## Table of Contents
1. [Core Concepts (Q1–Q20)](#1-core-concepts)
2. [Terraform Configuration (Q21–Q40)](#2-terraform-configuration)
3. [State Management (Q41–Q55)](#3-state-management)
4. [Modules & Workspaces (Q56–Q68)](#4-modules--workspaces)
5. [Terraform Commands (Q69–Q80)](#5-terraform-commands)
6. [Advanced & Real-World (Q81–Q110)](#6-advanced--real-world)

---

## 1. Core Concepts

**Q1. What is Terraform?**
> Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp. You write code to define infrastructure (servers, databases, networks) and Terraform creates, updates, and destroys it automatically.
> - Works with AWS, GCP, Azure, Kubernetes, and 1000+ providers
> - Language: HCL (HashiCorp Configuration Language)

---

**Q2. What is Infrastructure as Code (IaC)?**
> IaC means managing infrastructure using code files instead of manual clicks.
> Benefits:
> - **Repeatable:** Same code → same infrastructure every time
> - **Version controlled:** Track changes in Git
> - **Automated:** No manual steps
> - **Auditable:** Know who changed what, when
> - **Fast:** Spin up entire environments in minutes

---

**Q3. What is the difference between Terraform and Ansible?**
> | Feature | Terraform | Ansible |
> |---|---|---|
> | Purpose | Provision infrastructure | Configure software on servers |
> | Type | Declarative | Procedural (mostly) |
> | State | Maintains state file | Stateless |
> | Best for | Create EC2, VPC, RDS | Install nginx, deploy app |
> | Language | HCL | YAML (Playbooks) |
> Use both together: Terraform creates the server, Ansible configures it.

---

**Q4. What is the difference between Declarative and Imperative IaC?**
> - **Declarative (Terraform):** You say WHAT you want. Terraform figures out HOW.
>   `"I want 3 EC2 instances"` → Terraform creates them
> - **Imperative (scripts):** You say HOW to do it step by step.
>   `"Run this command, then this command, then..."`
> Declarative is easier to maintain and idempotent.

---

**Q5. What is HCL?**
> HCL (HashiCorp Configuration Language) is the language used to write Terraform code. It's human-readable and designed for infrastructure definitions.
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"

  tags = {
    Name = "WebServer"
    Env  = "Production"
  }
}
```

---

**Q6. What is a Terraform Provider?**
> A Provider is a plugin that lets Terraform interact with a specific platform (AWS, Azure, GCP, GitHub, Kubernetes, etc.). Each provider has its own resources and data sources.
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

---

**Q7. What is a Terraform Resource?**
> A Resource is the most important element — it defines a piece of infrastructure to create.
```hcl
resource "<provider>_<type>" "<local_name>" {
  # configuration
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-bucket-name"
}

resource "aws_instance" "web_server" {
  ami           = "ami-0abcdef"
  instance_type = "t3.micro"
}
```

---

**Q8. What is a Data Source in Terraform?**
> Data sources fetch information from existing infrastructure (not created by Terraform). Read-only.
```hcl
# Get latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Use it
resource "aws_instance" "web" {
  ami = data.aws_ami.amazon_linux.id
}
```

---

**Q9. What are Terraform Variables?**
> Variables make your code reusable and configurable:
```hcl
# Define variable
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "environment" {
  type    = string
  # no default - must be provided
}

# Use variable
resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

---

**Q10. What are Terraform Outputs?**
> Outputs expose values from your Terraform configuration — useful for sharing data between modules or displaying important info after apply.
```hcl
output "instance_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.web.public_ip
}

output "bucket_arn" {
  value = aws_s3_bucket.my_bucket.arn
}
```
```bash
terraform output instance_public_ip   # get specific output
terraform output -json                # all outputs as JSON
```

---

**Q11. What is the Terraform State file?**
> `terraform.tfstate` is a JSON file that tracks the current state of your infrastructure. It maps Terraform resources to real-world resources.
> - Terraform uses it to know what exists, what changed, what to create/destroy
> - Contains sensitive info (passwords, keys) — protect it!
> - Never edit it manually
> - Store remotely in production (S3 + DynamoDB)

---

**Q12. What is the Terraform workflow?**
> 4 main steps:
> ```
> terraform init    → Download providers and modules
> terraform plan    → Preview what will change (dry run)
> terraform apply   → Create/update/destroy infrastructure
> terraform destroy → Destroy all managed infrastructure
> ```

---

**Q13. What is `terraform init`?**
> Initializes a Terraform working directory:
> - Downloads provider plugins
> - Downloads modules
> - Sets up backend (remote state)
> Run this first in any new Terraform project or after adding providers.

---

**Q14. What is `terraform plan`?**
> Shows a preview of changes Terraform will make WITHOUT actually making them. Shows:
> - `+` resources to create
> - `~` resources to update in-place
> - `-` resources to destroy
> - `-/+` resources to destroy and recreate
> Always run plan before apply in production!

---

**Q15. What is idempotency in Terraform?**
> Running `terraform apply` multiple times with the same code produces the same result. If infrastructure already matches desired state, Terraform does nothing. This is safe and predictable.

---

**Q16. What are Terraform Locals?**
> Locals define computed values within a module — like variables but calculated, not input.
```hcl
locals {
  environment = "production"
  common_tags = {
    Environment = local.environment
    Project     = "MyApp"
    ManagedBy   = "Terraform"
  }
  bucket_name = "${local.environment}-myapp-data"
}

resource "aws_s3_bucket" "data" {
  bucket = local.bucket_name
  tags   = local.common_tags
}
```

---

**Q17. What is the difference between `variable`, `local`, and `output`?**
> - **variable:** Input values (passed in from outside)
> - **local:** Internal computed values (used within the module)
> - **output:** Exported values (shared with caller or displayed)

---

**Q18. What is Terraform Registry?**
> The Terraform Registry (`registry.terraform.io`) is a public repository of:
> - **Providers** — official and community providers
> - **Modules** — pre-built, reusable Terraform modules
> Example: `terraform-aws-modules/vpc/aws` is a popular VPC module.

---

**Q19. What is the `terraform.tfvars` file?**
> A file to set variable values without passing them on command line:
```hcl
# terraform.tfvars
environment    = "production"
instance_type  = "t3.medium"
instance_count = 3
region         = "ap-south-1"
```
> Terraform automatically loads `terraform.tfvars`. For other files: `terraform apply -var-file="prod.tfvars"`

---

**Q20. What is Terraform Cloud vs Terraform OSS?**
> - **Terraform OSS (Open Source):** Free, self-managed, local or custom backend
> - **Terraform Cloud:** HashiCorp's managed service — remote state, remote runs, team collaboration, policy enforcement (Sentinel), cost estimation
> - **Terraform Enterprise:** Self-hosted version of Terraform Cloud for enterprises

---

## 2. Terraform Configuration

**Q21. Write Terraform code to create a VPC with public and private subnets on AWS.**
```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "${var.project}-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags = { Name = "${var.project}-igw" }
}

# Public Subnets
resource "aws_subnet" "public" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = { Name = "${var.project}-public-${count.index + 1}" }
}

# Private Subnets
resource "aws_subnet" "private" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = var.availability_zones[count.index]

  tags = { Name = "${var.project}-private-${count.index + 1}" }
}

# NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
  tags = { Name = "${var.project}-nat" }
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  tags = { Name = "${var.project}-public-rt" }
}

resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```

---

**Q22. What is `count` in Terraform?**
> `count` creates multiple instances of a resource:
```hcl
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0abcdef"
  instance_type = "t3.micro"

  tags = {
    Name = "web-server-${count.index + 1}"
  }
}

# Reference: aws_instance.web[0], aws_instance.web[1], aws_instance.web[2]
```

---

**Q23. What is `for_each` in Terraform?**
> `for_each` creates resources from a map or set — each with a unique key:
```hcl
variable "buckets" {
  default = {
    logs    = "ap-south-1"
    backups = "us-east-1"
    static  = "ap-south-1"
  }
}

resource "aws_s3_bucket" "buckets" {
  for_each = var.buckets
  bucket   = "mycompany-${each.key}"

  tags = {
    Region = each.value
    Name   = each.key
  }
}
# Reference: aws_s3_bucket.buckets["logs"]
```

---

**Q24. What is the difference between `count` and `for_each`?**
> | Feature | count | for_each |
> |---|---|---|
> | Input | Number | Map or Set |
> | Reference | `[0]`, `[1]`, `[2]` | `["key"]` |
> | Deletion | Deletes by index (risky!) | Deletes by key (safe) |
> | Best for | Identical resources | Resources with unique properties |
> Prefer `for_each` — deleting middle item in `count` causes unintended replacements.

---

**Q25. What are Terraform expressions and functions?**
```hcl
# String interpolation
name = "myapp-${var.environment}"

# Conditional expression
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

# Functions
upper("hello")           # "HELLO"
lower("HELLO")           # "hello"
length(var.list)         # number of items
join(",", var.list)      # "a,b,c"
split(",", "a,b,c")      # ["a", "b", "c"]
toset(var.list)          # convert list to set
flatten([[1,2],[3,4]])   # [1,2,3,4]
cidrsubnet("10.0.0.0/16", 8, 0)  # "10.0.0.0/24"
file("script.sh")        # read file contents
```

---

**Q26. What is a `dynamic` block in Terraform?**
> Dynamic blocks generate repeated nested blocks programmatically:
```hcl
variable "ingress_rules" {
  default = [
    { port = 80,  protocol = "tcp" },
    { port = 443, protocol = "tcp" },
    { port = 22,  protocol = "tcp" }
  ]
}

resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

---

**Q27. What is the `lifecycle` block in Terraform?**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef"
  instance_type = "t3.micro"

  lifecycle {
    create_before_destroy = true  # create new before destroying old
    prevent_destroy       = true  # error if someone tries to destroy
    ignore_changes        = [tags, user_data]  # ignore these changes
    replace_triggered_by  = [aws_security_group.web]  # replace if SG changes
  }
}
```

---

**Q28. What is `depends_on` in Terraform?**
> Terraform automatically infers dependencies from references. `depends_on` adds explicit dependencies when there's no direct reference:
```hcl
resource "aws_iam_role_policy_attachment" "s3_access" {
  role       = aws_iam_role.app.name
  policy_arn = aws_iam_policy.s3.arn
}

resource "aws_instance" "app" {
  ami           = "ami-0abcdef"
  instance_type = "t3.micro"

  depends_on = [aws_iam_role_policy_attachment.s3_access]
  # Wait for IAM to be ready before creating EC2
}
```

---

**Q29. What are Terraform provisioners?**
> Provisioners run scripts on resources after creation. Use sparingly — prefer user_data, Ansible, or AMIs.
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef"
  instance_type = "t3.micro"

  # Run script on remote machine
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx"
    ]
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  # Run script on local machine
  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> inventory.txt"
  }
}
```

---

**Q30. What is `templatefile` function in Terraform?**
```hcl
# user_data.sh.tpl template file
#!/bin/bash
echo "Environment: ${environment}"
echo "DB Host: ${db_host}"
apt-get install -y ${packages}

# In Terraform
resource "aws_instance" "web" {
  user_data = templatefile("user_data.sh.tpl", {
    environment = var.environment
    db_host     = aws_db_instance.main.endpoint
    packages    = join(" ", ["nginx", "curl", "jq"])
  })
}
```

---

**Q31. What are Terraform type constraints?**
```hcl
variable "instance_count" {
  type    = number
  default = 2
}

variable "environment" {
  type    = string
}

variable "tags" {
  type = map(string)
  default = {}
}

variable "availability_zones" {
  type    = list(string)
  default = ["ap-south-1a", "ap-south-1b"]
}

variable "settings" {
  type = object({
    instance_type = string
    disk_size     = number
    enable_ha     = bool
  })
}
```

---

**Q32. What is the `moved` block in Terraform?**
> When you rename a resource or move it to a module, use `moved` to avoid destroy+recreate:
```hcl
moved {
  from = aws_instance.old_name
  to   = aws_instance.new_name
}

# Or moving into a module
moved {
  from = aws_instance.web
  to   = module.web_server.aws_instance.web
}
```

---

**Q33. What is `terraform import`?**
> Import existing infrastructure into Terraform state (without destroying and recreating it):
```bash
# Import existing S3 bucket
terraform import aws_s3_bucket.my_bucket my-existing-bucket-name

# Import existing EC2 instance
terraform import aws_instance.web i-1234567890abcdef0
```
> After import, you must write the Terraform config to match the existing resource.

---

**Q34. What is `terraform taint` and `terraform untaint`?**
> `terraform taint` marks a resource for forced recreation on next apply (even if config hasn't changed). Deprecated in newer versions — use `terraform apply -replace` instead.
```bash
# Modern way
terraform apply -replace="aws_instance.web"

# Old way (deprecated)
terraform taint aws_instance.web
```

---

**Q35. What are `precondition` and `postcondition` in Terraform?**
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    precondition {
      condition     = var.instance_type != "t1.micro"
      error_message = "t1.micro is too small, use at least t3.micro"
    }

    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must have a public IP"
    }
  }
}
```

---

**Q36. What is `terraform validate`?**
```bash
terraform validate
```
> Validates the Terraform configuration files for syntax errors and internal consistency. Does NOT check if values are valid (e.g., wrong AMI ID). Useful in CI/CD before plan.

---

**Q37. What is `terraform fmt`?**
```bash
terraform fmt           # format current directory
terraform fmt -recursive  # format all subdirectories
terraform fmt -check    # check formatting (exit 1 if not formatted) - use in CI
```
> Formats Terraform files to canonical style. Run in CI/CD to enforce consistent formatting.

---

**Q38. What is `terraform graph`?**
```bash
terraform graph | dot -Tsvg > graph.svg
```
> Generates a visual dependency graph of your infrastructure. Shows which resources depend on which.

---

**Q39. What are `null_resource` and `terraform_data`?**
```hcl
# null_resource runs provisioners without creating real infrastructure
resource "null_resource" "run_script" {
  triggers = {
    always_run = timestamp()  # run every apply
  }

  provisioner "local-exec" {
    command = "python update_dns.py ${aws_instance.web.public_ip}"
  }
}

# terraform_data (newer replacement for null_resource)
resource "terraform_data" "bootstrap" {
  input = aws_instance.web.public_ip
  provisioner "local-exec" {
    command = "ansible-playbook -i ${self.output} playbook.yml"
  }
}
```

---

**Q40. What is the `random` provider in Terraform?**
```hcl
resource "random_string" "bucket_suffix" {
  length  = 8
  special = false
  upper   = false
}

resource "aws_s3_bucket" "data" {
  bucket = "myapp-${random_string.bucket_suffix.result}"
  # Creates: myapp-a1b2c3d4
}

resource "random_password" "db_password" {
  length           = 16
  special          = true
  override_special = "!#$%^&*"
}
```

---

## 3. State Management

**Q41. What is Terraform State and why is it important?**
> The state file (`terraform.tfstate`) is Terraform's record of what infrastructure exists. It:
> - Maps Terraform resources to real-world IDs
> - Tracks metadata needed for updates
> - Determines what changes are needed (diff between desired and actual)
> Without state, Terraform can't manage existing infrastructure.

---

**Q42. What is a Remote Backend?**
> Store state file remotely instead of locally. Required for team collaboration.
```hcl
terraform {
  backend "s3" {
    bucket         = "mycompany-terraform-state"
    key            = "production/vpc/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```
> Popular backends: S3 (AWS), GCS (GCP), Azure Blob, Terraform Cloud, HashiCorp Consul.

---

**Q43. What is State Locking?**
> State locking prevents multiple people from running `terraform apply` at the same time (which would corrupt state).
> - **S3 backend:** Uses DynamoDB table for locking
> - **Terraform Cloud:** Built-in locking
> - Lock is acquired on plan/apply, released when done
> If process crashes: `terraform force-unlock <lock-id>`

---

**Q44. Why should you never store state in Git?**
> 1. State contains **sensitive data** (passwords, keys) in plain text
> 2. Git merge conflicts on state files are destructive
> 3. No locking — two people apply simultaneously = corruption
> Always use remote backend (S3, Terraform Cloud).

---

**Q45. What is `terraform state` command?**
```bash
terraform state list                    # list all resources in state
terraform state show aws_instance.web   # show details of resource
terraform state mv aws_instance.old aws_instance.new  # rename resource
terraform state rm aws_instance.web     # remove from state (doesn't destroy)
terraform state pull                    # download remote state locally
terraform state push                    # upload local state to remote
```

---

**Q46. What is the difference between `terraform destroy` and `terraform state rm`?**
> - `terraform destroy` — Destroys real infrastructure AND removes from state
> - `terraform state rm` — Removes from state ONLY. Real infrastructure still exists.
> Use `state rm` when you want Terraform to "forget" a resource without destroying it.

---

**Q47. What is a partial configuration in Terraform backend?**
> Backend config can be split between the code and runtime flags (for security):
```hcl
# backend.tf - code (no sensitive values)
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    region = "ap-south-1"
    # key provided at init time
  }
}
```
```bash
# Pass sensitive values at init
terraform init -backend-config="key=prod/app/terraform.tfstate" \
               -backend-config="dynamodb_table=terraform-locks"
```

---

**Q48. What is a state file refresh?**
```bash
terraform refresh   # deprecated, use:
terraform apply -refresh-only
```
> Updates state file to match actual real-world infrastructure (in case someone made manual changes). Doesn't change infrastructure — only updates state.

---

**Q49. How do you handle Terraform state for multiple environments?**
> **Option 1: Separate state files per environment (recommended)**
```
terraform/
├── environments/
│   ├── dev/
│   │   └── main.tf  (backend key: dev/terraform.tfstate)
│   ├── staging/
│   │   └── main.tf  (backend key: staging/terraform.tfstate)
│   └── prod/
│       └── main.tf  (backend key: prod/terraform.tfstate)
```
> **Option 2: Workspaces** (for simple setups)
> **Option 3: Terragrunt** (DRY wrapper for Terraform)

---

**Q50. What is a Terraform state drift?**
> Drift happens when real infrastructure differs from Terraform state — because someone made manual changes (clicked in AWS console, ran AWS CLI).
```bash
# Detect drift
terraform plan  # shows differences

# Fix drift options:
# 1. Apply to revert to Terraform's desired state
terraform apply

# 2. Update state to match reality
terraform apply -refresh-only

# 3. Import the changed resource
terraform import aws_instance.web i-1234567890
```

---

**Q51. What is `terraform output` and how to use it in scripts?**
```bash
terraform output                          # show all outputs
terraform output instance_ip              # specific output
terraform output -raw instance_ip         # raw value (no quotes)
terraform output -json                    # all as JSON

# Use in shell script
INSTANCE_IP=$(terraform output -raw instance_ip)
ssh ubuntu@$INSTANCE_IP
```

---

**Q52. How do you share state between Terraform configurations?**
> Use `terraform_remote_state` data source:
```hcl
# In app config - read VPC state from network config
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "mycompany-terraform-state"
    key    = "production/network/terraform.tfstate"
    region = "ap-south-1"
  }
}

# Use outputs from network state
resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.network.outputs.private_subnet_id
}
```

---

**Q53. What is sensitive output in Terraform?**
```hcl
output "db_password" {
  value     = aws_db_instance.main.password
  sensitive = true  # won't show in terminal output
}
```
> Sensitive outputs are redacted in logs and terminal but still stored in state file (in plain text).

---

**Q54. How do you encrypt Terraform state?**
> **S3 backend:**
```hcl
backend "s3" {
  bucket  = "my-terraform-state"
  encrypt = true  # server-side encryption with S3-managed keys
  kms_key_id = "arn:aws:kms:..."  # or use KMS key
}
```
> **etcd/Consul:** Configure encryption at rest.
> **Terraform Cloud:** Encrypted by default.

---

**Q55. What happens if two people run `terraform apply` simultaneously?**
> - **With state locking (DynamoDB):** Second person gets error: "Error acquiring the state lock". They must wait.
> - **Without locking:** Both apply simultaneously → state file gets corrupted → infrastructure inconsistency.
> Always use remote backend with locking for team environments!

---

## 4. Modules & Workspaces

**Q56. What is a Terraform Module?**
> A Module is a reusable package of Terraform code. It's a directory of `.tf` files. Every Terraform config is a module (the root module). You can call child modules:
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
  azs  = ["ap-south-1a", "ap-south-1b", "ap-south-1c"]

  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
}
```

---

**Q57. What are Module Sources?**
```hcl
# Local path
module "security" {
  source = "./modules/security"
}

# Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
}

# GitHub
module "app" {
  source = "github.com/myorg/terraform-modules//app?ref=v1.2.0"
}

# S3
module "network" {
  source = "s3::https://s3-ap-south-1.amazonaws.com/my-modules/network.zip"
}
```

---

**Q58. How do you structure a Terraform module?**
```
modules/
└── ec2-instance/
    ├── main.tf         # resources
    ├── variables.tf    # input variables
    ├── outputs.tf      # output values
    ├── versions.tf     # required providers/versions
    └── README.md       # documentation
```

---

**Q59. What is module versioning?**
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"    # exact version
  # version = "~> 5.0" # allow 5.x but not 6.x
  # version = ">= 4.0, < 6.0"  # range
}
```
> Always pin module versions in production to avoid unexpected changes.

---

**Q60. What is a Terraform Workspace?**
> Workspaces allow multiple state files within the same backend configuration. Like branches for infrastructure state.
```bash
terraform workspace new dev         # create workspace
terraform workspace new prod
terraform workspace list            # list workspaces
terraform workspace select prod     # switch workspace
terraform workspace show            # current workspace
terraform workspace delete dev      # delete workspace
```
```hcl
# Use workspace name in config
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
}
```

---

**Q61. When should you use Workspaces vs separate directories?**
> - **Workspaces:** Good for temporary environments, same config with minor differences
> - **Separate directories:** Better for very different environments (dev vs prod), clearer separation, easier to have different configs
> Many teams avoid workspaces — separate directories with separate state files are safer.

---

**Q62. What is Terragrunt?**
> Terragrunt is a thin wrapper around Terraform that adds:
> - DRY (Don't Repeat Yourself) — share backend config, providers across environments
> - Dependency management between Terraform modules
> - Easier multi-account, multi-region setups
```
├── terragrunt.hcl        # root config (backend, provider)
├── dev/
│   ├── vpc/
│   │   └── terragrunt.hcl
│   └── app/
│       └── terragrunt.hcl
└── prod/
    ├── vpc/
    │   └── terragrunt.hcl
    └── app/
        └── terragrunt.hcl
```

---

**Q63. What is a Terraform Module Registry (private)?**
> Teams can host private module registries in:
> - **Terraform Cloud/Enterprise** — built-in private registry
> - **GitHub/GitLab** — modules as repos with tags
> - **Artifactory** — enterprise artifact management

---

**Q64. How do you pass outputs from one module to another?**
```hcl
module "network" {
  source = "./modules/network"
}

module "app" {
  source    = "./modules/app"
  vpc_id    = module.network.vpc_id        # pass network output to app
  subnet_ids = module.network.private_subnet_ids
}
```

---

**Q65. What is module composition?**
> Building complex infrastructure by combining smaller, focused modules:
```hcl
# Root module composes everything
module "network"  { source = "./modules/network" }
module "database" { source = "./modules/database"
                    vpc_id = module.network.vpc_id }
module "app"      { source = "./modules/app"
                    vpc_id = module.network.vpc_id
                    db_url = module.database.endpoint }
module "monitoring" { source = "./modules/monitoring"
                      app_ids = module.app.instance_ids }
```

---

**Q66. What is the `versions.tf` file convention?**
```hcl
# versions.tf
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }
}
```
> Best practice: Always pin Terraform and provider versions.

---

**Q67. What is `terraform providers lock`?**
```bash
terraform providers lock \
  -platform=linux_amd64 \
  -platform=darwin_arm64
```
> Creates `.terraform.lock.hcl` — locks exact provider versions and checksums. Commit this to Git so all team members use same provider versions.

---

**Q68. What is the `.terraform.lock.hcl` file?**
> Lock file that records exact provider versions and checksums. Similar to `package-lock.json` in Node.js.
> - **Always commit to Git**
> - Ensures reproducible builds across team and CI/CD
> - Updated with `terraform init -upgrade`

---

## 5. Terraform Commands

**Q69. Complete Terraform command reference.**
```bash
# Setup
terraform init                    # initialize
terraform init -upgrade           # upgrade providers
terraform init -reconfigure       # force backend reconfigure

# Planning
terraform plan                    # show changes
terraform plan -out=tfplan        # save plan to file
terraform plan -target=aws_instance.web  # plan specific resource
terraform plan -var="env=prod"    # pass variable
terraform plan -var-file=prod.tfvars

# Applying
terraform apply                   # apply (with confirmation)
terraform apply -auto-approve     # skip confirmation (CI/CD)
terraform apply tfplan            # apply saved plan
terraform apply -target=aws_instance.web  # apply specific resource
terraform apply -replace=aws_instance.web # force replace resource

# Destroying
terraform destroy                 # destroy all
terraform destroy -target=aws_instance.web  # destroy specific
terraform destroy -auto-approve   # skip confirmation

# State
terraform state list
terraform state show <resource>
terraform state mv <from> <to>
terraform state rm <resource>
terraform state pull
terraform state push

# Other
terraform fmt -recursive          # format code
terraform validate                # validate syntax
terraform output                  # show outputs
terraform import <resource> <id>  # import existing
terraform refresh                 # sync state with reality
terraform graph                   # show dependency graph
terraform workspace <cmd>         # manage workspaces
terraform force-unlock <lock-id>  # unlock stuck state
```

---

**Q70. How do you run Terraform in CI/CD pipeline?**
```yaml
# GitHub Actions example
- name: Terraform Init
  run: terraform init
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

- name: Terraform Format Check
  run: terraform fmt -check -recursive

- name: Terraform Validate
  run: terraform validate

- name: Terraform Plan
  run: terraform plan -out=tfplan -no-color
  
- name: Terraform Apply (main branch only)
  if: github.ref == 'refs/heads/main'
  run: terraform apply -auto-approve tfplan
```

---

**Q71. How do you target specific resources in Terraform?**
```bash
# Plan/apply only specific resource
terraform plan -target=aws_instance.web
terraform apply -target=module.database

# Use case: fix one broken resource without affecting others
terraform apply -target=aws_security_group.app -auto-approve
```
> Use `-target` sparingly — it can cause state inconsistencies.

---

**Q72. What is `terraform console`?**
```bash
terraform console
# Interactive REPL for testing expressions

> cidrsubnet("10.0.0.0/16", 8, 1)
"10.0.1.0/24"

> length(["a", "b", "c"])
3

> upper("hello")
"HELLO"

> var.environment  # if terraform.tfvars loaded
"production"
```

---

**Q73. How do you use Terraform with multiple AWS accounts?**
```hcl
# Using AWS profiles
provider "aws" {
  alias   = "dev"
  profile = "dev-account"
  region  = "ap-south-1"
}

provider "aws" {
  alias   = "prod"
  profile = "prod-account"
  region  = "ap-south-1"
}

resource "aws_instance" "dev_server" {
  provider = aws.dev
  ami      = "ami-0abcdef"
}

# Using assume_role
provider "aws" {
  alias = "prod"
  assume_role {
    role_arn = "arn:aws:iam::PROD_ACCOUNT:role/TerraformRole"
  }
}
```

---

**Q74. What is `terraform show`?**
```bash
terraform show              # show current state in human-readable format
terraform show tfplan       # show contents of a saved plan file
terraform show -json        # output as JSON
```

---

**Q75. How do you debug Terraform?**
```bash
# Enable detailed logging
export TF_LOG=DEBUG         # DEBUG, INFO, WARN, ERROR, TRACE
export TF_LOG_PATH=./terraform.log

terraform plan 2>&1 | tee plan.log

# Check provider version
terraform version
terraform providers

# Validate without providers
terraform validate
```

---

**Q76. What is `-parallelism` flag in Terraform?**
```bash
terraform apply -parallelism=20  # default is 10
```
> Controls how many resource operations run in parallel. Increase for faster applies in large environments. Decrease if hitting API rate limits.

---

**Q77. What is `terraform workspace` used for in practice?**
```bash
# Create environment-specific deployments
terraform workspace new feature-branch-123
terraform workspace select feature-branch-123
terraform apply -var="environment=dev"

# Clean up after PR merge
terraform destroy -auto-approve
terraform workspace select default
terraform workspace delete feature-branch-123
```

---

**Q78. How do you handle secrets in Terraform?**
> 1. Use environment variables: `export TF_VAR_db_password=secret`
> 2. Use AWS Secrets Manager / Parameter Store as data source
> 3. Use HashiCorp Vault provider
> 4. Mark variables as sensitive: `sensitive = true`
> 5. Never put secrets in `.tfvars` files committed to Git
```hcl
# Fetch from AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_pass" {
  secret_id = "prod/myapp/db-password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_pass.secret_string
}
```

---

**Q79. What is `terraform force-unlock`?**
```bash
terraform force-unlock LOCK_ID
```
> Manually releases a stuck state lock. Use only if you're sure no other operation is running. Lock ID is shown in the error message when lock acquisition fails.

---

**Q80. How do you test Terraform code?**
> Testing tools:
> - **terraform validate** — syntax check
> - **terraform plan** — dry run
> - **Terratest** — Go-based testing framework, actually deploys and tests
> - **terraform test** — built-in testing (Terraform 1.6+)
> - **Checkov** — static analysis for security issues
> - **tflint** — linting for Terraform
> - **Sentinel** — policy as code (Terraform Cloud)

---

## 6. Advanced & Real-World

**Q81. How do you structure a large Terraform project?**
```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   └── prod/
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── eks/
    ├── rds/
    └── app/
```

---

**Q82. How do you handle Terraform code for 100+ resources without it becoming a mess?**
> 1. **Split into modules** — each module handles one concern (VPC, EKS, RDS)
> 2. **Separate state files** — network state, app state, database state
> 3. **Use Terragrunt** — manage dependencies between configs
> 4. **Remote state references** — `terraform_remote_state` for cross-config data
> 5. **Clear naming conventions** — `<project>-<environment>-<resource>`

---

**Q83. What is Sentinel in Terraform?**
> Sentinel is HashiCorp's Policy as Code framework for Terraform Cloud/Enterprise. You write policies that must pass before `apply` is allowed.
```python
# Example Sentinel policy
import "tfplan/v2" as tfplan

# Require all EC2 instances to have specific tags
main = rule {
  all tfplan.resource_changes as _, changes {
    changes.type != "aws_instance" or
    "Environment" in changes.change.after.tags
  }
}
```

---

**Q84. How do you implement blue-green infrastructure with Terraform?**
```hcl
variable "active_color" {
  default = "blue"  # switch to "green" for blue-green swap
}

resource "aws_autoscaling_group" "blue" {
  count = var.active_color == "blue" ? 1 : 0
  # ... blue config
}

resource "aws_autoscaling_group" "green" {
  count = var.active_color == "green" ? 1 : 0
  # ... green config
}

resource "aws_lb_listener_rule" "main" {
  # Route to active color's target group
  action {
    target_group_arn = var.active_color == "blue" ?
      aws_lb_target_group.blue.arn :
      aws_lb_target_group.green.arn
  }
}
```

---

**Q85. What is `terraform plan -generate-config-out`?**
```bash
# Terraform 1.5+ - generate config from imported resources
terraform plan -generate-config-out=generated.tf
```
> Automatically generates Terraform configuration for imported resources. Replaces manual config writing after `terraform import`.

---

**Q86. How do you manage Terraform at scale (many teams)?**
> 1. **Module library** — shared, versioned modules for all teams
> 2. **Terraform Cloud/Enterprise** — centralized state, runs, policies
> 3. **Atlantis** — open source GitOps for Terraform (PR automation)
> 4. **Terragrunt** — DRY wrapper across many configs
> 5. **OPA/Sentinel** — policy enforcement
> 6. **Module versioning** — semantic versioning for modules

---

**Q87. What is Atlantis?**
> Atlantis is an open-source tool for Terraform Pull Request automation:
> - Comment `atlantis plan` on a PR → runs `terraform plan` and posts results
> - Comment `atlantis apply` on an approved PR → runs `terraform apply`
> - Enforces code review before apply
> - Eliminates the need for engineers to run Terraform locally

---

**Q88. What is `terraform apply -refresh=false`?**
```bash
terraform apply -refresh=false
```
> Skips refreshing state from real infrastructure before applying. Faster for large environments. Use only when you're sure no manual changes were made.

---

**Q89. How do you use Terraform with EKS?**
```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "my-cluster"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    main = {
      instance_types = ["t3.medium"]
      min_size       = 1
      max_size       = 10
      desired_size   = 3
    }
  }
}

# Configure kubectl after cluster creation
output "kubeconfig_command" {
  value = "aws eks update-kubeconfig --name ${module.eks.cluster_name} --region ap-south-1"
}
```

---

**Q90. What is the `locals` block and when to use it?**
```hcl
locals {
  # Compute once, use many times
  name_prefix = "${var.project}-${var.environment}"

  common_tags = merge(var.extra_tags, {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
    CreatedAt   = timestamp()
  })

  # Complex computations
  azs = slice(data.aws_availability_zones.available.names, 0, 3)
}

resource "aws_vpc" "main" {
  tags = local.common_tags
}
```

---

**Q91. How do you implement auto-scaling with Terraform?**
```hcl
resource "aws_autoscaling_group" "app" {
  name                = "${local.name_prefix}-asg"
  vpc_zone_identifier = module.vpc.private_subnet_ids
  target_group_arns   = [aws_lb_target_group.app.arn]
  min_size            = 2
  max_size            = 20
  desired_capacity    = 2
  health_check_type   = "ELB"

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }
}

resource "aws_autoscaling_policy" "scale_up" {
  name                   = "scale-up"
  autoscaling_group_name = aws_autoscaling_group.app.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 70.0
  }
}
```

---

**Q92. What is the `check` block in Terraform?**
```hcl
# Terraform 1.5+ - validation checks that run after apply
check "website_is_up" {
  data "http" "myapp" {
    url = "https://${aws_lb.main.dns_name}/health"
  }

  assert {
    condition     = data.http.myapp.status_code == 200
    error_message = "Website is not responding with 200"
  }
}
```

---

**Q93. How do you use Terraform with Kubernetes?**
```hcl
provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)

  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
  }
}

resource "kubernetes_namespace" "production" {
  metadata {
    name = "production"
    labels = {
      "pod-security.kubernetes.io/enforce" = "restricted"
    }
  }
}

resource "kubernetes_deployment" "app" {
  metadata {
    name      = "myapp"
    namespace = kubernetes_namespace.production.metadata[0].name
  }
  spec {
    replicas = 3
    # ... spec
  }
}
```

---

**Q94. What is CDKTF?**
> CDKTF (Cloud Development Kit for Terraform) lets you write Terraform infrastructure using TypeScript, Python, Java, Go, or C# — instead of HCL. Generates Terraform JSON configuration under the hood.
```python
from cdktf import App, TerraformStack
from constructs import Construct
from cdktf_cdktf_provider_aws.instance import Instance

class MyStack(TerraformStack):
    def __init__(self, scope, id):
        super().__init__(scope, id)
        Instance(self, "web",
                 ami="ami-0abcdef",
                 instance_type="t3.micro")
```

---

**Q95. What are common Terraform mistakes and how to avoid them?**
> 1. ❌ Storing state in Git → ✅ Use remote backend
> 2. ❌ Not pinning provider versions → ✅ Pin with `version = "~> 5.0"`
> 3. ❌ Hardcoding values → ✅ Use variables and locals
> 4. ❌ One huge `main.tf` → ✅ Split into modules
> 5. ❌ No state locking → ✅ Use DynamoDB with S3 backend
> 6. ❌ Running apply without plan → ✅ Always plan first in prod
> 7. ❌ Secrets in `.tfvars` → ✅ Use environment variables or Secrets Manager
> 8. ❌ Not using `-target` carefully → ✅ Understand it causes partial state

---

**Q96. How do you do cost estimation with Terraform?**
> Tools:
> - **Infracost** — open source, shows cost breakdown in PR comments
> - **Terraform Cloud** — built-in cost estimation
```bash
# Infracost
infracost breakdown --path .
infracost diff --path . --compare-to previous-plan.json
```
> Integrate in CI/CD to see cost impact of every infrastructure change.

---

**Q97. What is the difference between `terraform apply` and `terraform push`?**
> `terraform push` was used with Atlas (old Terraform Enterprise) — it's **deprecated**.
> Modern equivalent: `terraform apply` with Terraform Cloud remote execution, or push code to Git and let Atlantis/Terraform Cloud handle it.

---

**Q98. How do you handle Terraform for disaster recovery?**
> 1. Store state in S3 with cross-region replication
> 2. Enable S3 versioning for state file history
> 3. Use multiple providers for multi-region resources
```hcl
provider "aws" {
  alias  = "primary"
  region = "ap-south-1"
}

provider "aws" {
  alias  = "dr"
  region = "us-east-1"
}

resource "aws_s3_bucket" "primary" {
  provider = aws.primary
  bucket   = "myapp-primary"
}

resource "aws_s3_bucket_replication_configuration" "dr" {
  provider = aws.primary
  # ... replicate to DR region
}
```

---

**Q99. What is `terraform apply -json`?**
```bash
terraform plan -out=tfplan
terraform show -json tfplan > plan.json
# Parse plan.json in CI to check what's changing
```
> Machine-readable JSON output. Used in CI/CD for automated plan analysis, cost estimation, policy checks.

---

**Q100. Real-world scenario: Complete Terraform setup for a production AWS environment.**
```
Project Structure:
terraform/
├── modules/
│   ├── vpc/          (VPC, subnets, NAT, IGW)
│   ├── eks/          (EKS cluster, node groups)
│   ├── rds/          (RDS PostgreSQL, Multi-AZ)
│   ├── elasticache/  (Redis cluster)
│   └── security/     (Security groups, IAM roles)
├── environments/
│   ├── dev/
│   │   ├── main.tf   (calls modules, small sizes)
│   │   ├── vars.tf
│   │   └── backend.tf (S3: dev/terraform.tfstate)
│   └── prod/
│       ├── main.tf   (calls modules, prod sizes)
│       ├── vars.tf
│       └── backend.tf (S3: prod/terraform.tfstate)

CI/CD Flow:
PR opened         → terraform fmt -check, validate, plan
PR approved       → atlantis apply (dev)
Merge to main     → terraform apply (staging auto)
Manual approval   → terraform apply (production)
```

---

**Q101–Q110. Quick-fire important questions.**

**Q101. What is `terraform graph` used for?**
> Generates a DOT format dependency graph. Visualize resource dependencies for debugging complex configurations.

**Q102. What is the difference between `~>` and `>=` in version constraints?**
> - `~> 5.0` means `>= 5.0, < 6.0` (allows patch/minor updates within major)
> - `~> 5.2.1` means `>= 5.2.1, < 5.3.0` (allows patch updates only)
> - `>= 5.0` means any version 5.0 or higher (too permissive for production)

**Q103. What is `terraform init -backend=false`?**
> Initializes without configuring the backend. Useful for syntax validation in CI when you don't have backend credentials.

**Q104. What is the `ignore_changes` lifecycle argument used for?**
> Tells Terraform to ignore certain attribute changes made outside of Terraform (e.g., auto-scaling changes `desired_capacity`, don't revert it).

**Q105. What is a Terraform ephemeral value? (v1.10+)**
> Ephemeral values exist only during a Terraform run and are never stored in state or plan files. Used for truly sensitive values like passwords.

**Q106. What does `create_before_destroy = true` do?**
> Creates the replacement resource BEFORE destroying the old one. Useful for zero-downtime replacements (e.g., new SSL certificate before deleting old one).

**Q107. What is `terraform apply -destroy`?**
> Same as `terraform destroy`. Generates a destroy plan and applies it.

**Q108. What is the Terraform `for` expression?**
```hcl
# Transform a list
output "upper_names" {
  value = [for name in var.names : upper(name)]
}
# Filter a list
output "prod_instances" {
  value = [for i in var.instances : i if i.env == "prod"]
}
# Create a map
output "name_to_id" {
  value = {for inst in var.instances : inst.name => inst.id}
}
```

**Q109. What is `tostring`, `tonumber`, `tobool` in Terraform?**
> Type conversion functions: `tostring(42)` → `"42"`, `tonumber("42")` → `42`, `tobool("true")` → `true`. Used when types don't match.

**Q110. What is the recommended Terraform project file structure?**
```
main.tf        # main resources
variables.tf   # input variable declarations
outputs.tf     # output value declarations
versions.tf    # required terraform/provider versions
locals.tf      # local value computations
data.tf        # data source lookups
backend.tf     # backend configuration
```

---
---
