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

# terraform validate
jeenicj@DESKTOP-BG3MAVI:~/terra-day1$ terraform validate
Success! The configuration is valid.

# terraform plan
jeenicj@DESKTOP-BG3MAVI:~/terra-day1$ terraform plan

Terraform used the selected providers to generate the following
execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # local_file.greeting will be created
  + resource "local_file" "greeting" {
      + content              = (known after apply)
      + content_base64sha256 = (known after apply)
      + content_base64sha512 = (known after apply)
      + content_md5          = (known after apply)
      + content_sha1         = (known after apply)
      + content_sha256       = (known after apply)
      + content_sha512       = (known after apply)
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "./greeting.txt"
      + id                   = (known after apply)
    }

  # random_pet.name will be created
  + resource "random_pet" "name" {
      + id        = (known after apply)
      + length    = 2
      + separator = "-"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + file_path = "./greeting.txt"
  + pet_name  = (known after apply)

# terraform apply
jeenicj@DESKTOP-BG3MAVI:~/terra-day1$ terraform apply

Terraform used the selected providers to generate the following
execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # local_file.greeting will be created
  + resource "local_file" "greeting" {
      + content              = (known after apply)
      + content_base64sha256 = (known after apply)
      + content_base64sha512 = (known after apply)
      + content_md5          = (known after apply)
      + content_sha1         = (known after apply)
      + content_sha256       = (known after apply)
      + content_sha512       = (known after apply)
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "./greeting.txt"
      + id                   = (known after apply)
    }

  # random_pet.name will be created
  + resource "random_pet" "name" {
      + id        = (known after apply)
      + length    = 2
      + separator = "-"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + file_path = "./greeting.txt"
  + pet_name  = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

random_pet.name: Creating...
random_pet.name: Creation complete after 0s [id=growing-bull]
local_file.greeting: Creating...
local_file.greeting: Creation complete after 0s [id=436c0e46caf12448b0027ec0246bef8823833753]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

file_path = "./greeting.txt"
pet_name = "growing-bull"

# cat greeting.txt
jeenicj@DESKTOP-BG3MAVI:~/terra-day1$ cat greeting.txt
Hello from TerraWeek 2026! 🚀
Your infra pet name is: growing-bull

# terraform destroy
jeenicj@DESKTOP-BG3MAVI:~/terra-day1$ terraform destroy
random_pet.name: Refreshing state... [id=growing-bull]
local_file.greeting: Refreshing state... [id=436c0e46caf12448b0027ec0246bef8823833753]

Terraform used the selected providers to generate the following
execution plan. Resource actions are indicated with the following
symbols:
  - destroy

Terraform will perform the following actions:

  # local_file.greeting will be destroyed
  - resource "local_file" "greeting" {
      - content              = <<-EOT
            Hello from TerraWeek 2026! 🚀
            Your infra pet name is: growing-bull
        EOT -> null
      - content_base64sha256 = "q2Z/xuHcOm7HcsIBm2tRMjxnaijfdhDFDDZSQnoKa4Q=" -> null
      - content_base64sha512 = "FWuQsVzQRacUR4bZpv1qPpwfckemlYdIJXj3hrOUQkeYaRR1J9j2exxfpYMQxFkheRHVfv2wNInFITH1NaTw7g==" -> null
      - content_md5          = "8b401fc34c092949983d0cf8b045ab5d" -> null
      - content_sha1         = "436c0e46caf12448b0027ec0246bef8823833753" -> null
      - content_sha256       = "ab667fc6e1dc3a6ec772c2019b6b51323c676a28df7610c50c3652427a0a6b84" -> null
      - content_sha512       = "156b90b15cd045a7144786d9a6fd6a3e9c1f7247a69587482578f786b39442479869147527d8f67b1c5fa58310c459217911d57efdb03489c52131f535a4f0ee" -> null
      - directory_permission = "0777" -> null
      - file_permission      = "0777" -> null
      - filename             = "./greeting.txt" -> null
      - id                   = "436c0e46caf12448b0027ec0246bef8823833753" -> null
    }

  # random_pet.name will be destroyed
  - resource "random_pet" "name" {
      - id        = "growing-bull" -> null
      - length    = 2 -> null
      - separator = "-" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Changes to Outputs:
  - file_path = "./greeting.txt" -> null
  - pet_name  = "growing-bull" -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

local_file.greeting: Destroying... [id=436c0e46caf12448b0027ec0246bef8823833753]
local_file.greeting: Destruction complete after 0s
random_pet.name: Destroying... [id=growing-bull]
random_pet.name: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
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
