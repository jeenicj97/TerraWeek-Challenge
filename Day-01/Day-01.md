# 🌱 TerraWeek Day 1 — Introduction to IaC & Terraform Basics

**Date:** Sunday, 12th July 2026

Welcome to **Day 1** of the TerraWeek Challenge! Today is all about **foundations** — understanding *why* Infrastructure as Code exists, installing the **latest Terraform (v1.15.x)**, and running your very first `terraform apply`. 🚀

---

## Task 1: Understanding IaC & Terraform

### What is Infrastructure as Code (IaC)?

IaC means writing code to create and manage infrastructure instead of manually clicking through a cloud console.

**Problems it solves:**

| Without IaC (ClickOps) | With IaC |
|---|---|
| Manual steps, easy to make mistakes | Automated, consistent every time |
| Hard to repeat the same setup | Run the same code anywhere |
| No record of what changed | Git tracks every change |
| Only one person knows how it was set up | Anyone can read the code |

---

### What is Terraform and why is it popular?

Terraform is an open-source tool by HashiCorp that lets you create and manage infrastructure using code.

**Why everyone uses it:**

- **Declarative** — you say *what* you want, Terraform figures out *how* to do it
- **Provider-agnostic** — works with AWS, Azure, GCP, Kubernetes and 1000+ more
- **Plan before apply** — shows you exactly what will change before doing anything
- **State management** — keeps track of what it created so it knows what to update or delete
- **Huge ecosystem** — thousands of ready-made modules on Terraform Registry

---

### Terraform vs Alternatives

| Tool | Comparison |
|---|---|
| **OpenTofu** | Open-source fork of Terraform — almost identical, community-driven |
| **Pulumi** | Like Terraform but you write in Python/TypeScript instead of HCL |
| **CloudFormation** | AWS only — works great on AWS but locked to one cloud |
| **Ansible** | Configures existing servers (installs software etc), doesn't provision infrastructure |

> Terraform builds the house. Ansible furnishes it. Kubernetes runs apps inside it.

---

## Task 2: Install Terraform

**Output:**
```bash
jeenicj@DESKTOP-BG3MAVI:/mnt/c/Users/Jeeni$ terraform version
Terraform v1.15.8
on linux_amd64

jeenicj@DESKTOP-BG3MAVI:/mnt/c/Users/Jeeni$ terraform -help
Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure
```

---

## Task 3: 6 Key Terraform Terminologies

### 1. Provider
A plugin that tells Terraform which platform to talk to.
```hcl
provider "azurerm" {
  features {}
}
```

### 2. Resource
A piece of infrastructure you want to create.
```hcl
resource "azurerm_resource_group" "rg" {
  name     = "my-rg"
  location = "East US"
}
```

### 3. State
Terraform's record of what it created — stored in `terraform.tfstate`.
Terraform reads this file to know what already exists and what needs to change.

### 4. Plan
A preview of what Terraform will do before it does anything.
```bash
terraform plan
# shows: what will be created (+), updated (~), or destroyed (-)
```

### 5. HCL
HashiCorp Configuration Language — the syntax you write Terraform files in.
Human-readable, looks like JSON but cleaner.

### 6. Module
A reusable group of Terraform files packaged together.
Instead of writing the same AKS config every project, use a module once.
```hcl
module "aks" {
  source = "Azure/aks/azurerm"
}
```

---

## Task 4: Core Terraform Workflow

### The Workflow

```
Write  ──▶  Init  ──▶  Plan  ──▶  Apply  ──▶  Destroy
(.tf)      (setup)   (preview)  (create)   (clean up)
```

### Commands and what they do

```bash
terraform init      # downloads providers, sets up working directory
terraform fmt       # auto-formats your .tf files
terraform validate  # checks for syntax errors
terraform plan      # previews what will be created/changed/destroyed
terraform apply     # actually creates the resources (type: yes)
terraform destroy   # deletes everything Terraform created (type: yes)
```

### Output from hands-on run

```bash
# terraform init
jeenicj@DESKTOP-BG3MAVI:~/terra-day1$ terraform init
Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/local versions matching "~> 2.5"...
- Finding hashicorp/random versions matching "~> 3.7"...
- Installing hashicorp/local v2.9.0...
- Installed hashicorp/local v2.9.0 (signed by HashiCorp)
- Installing hashicorp/random v3.9.0...
- Installed hashicorp/random v3.9.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!


# terraform plan
# paste output here

# terraform apply
# paste output here

# cat greeting.txt
# paste output here

# terraform destroy
# paste output here
```

---

## Bonus

### Tab completion
```bash
terraform -install-autocomplete
```

### .terraform.lock.hcl — what is it?
- Created automatically after `terraform init`
- Records the exact version of each provider that was downloaded
- Should be committed to Git so everyone on the team uses the same provider versions
- Similar to `package-lock.json` in Node.js

### OpenTofu
- Open-source fork of Terraform created in 2023
- HashiCorp changed Terraform's licence from open-source to BSL
- OpenTofu kept the original open-source licence
- Commands are identical — `tofu init`, `tofu plan`, `tofu apply`
- Differences are minimal right now but may grow over time

---

*Source: TerraWeek Challenge — Day 1*
