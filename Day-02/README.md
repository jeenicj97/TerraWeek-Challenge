## 🧩 TerraWeek Day 2 - HCL Deep Dive: Variables, Types & Expressions

**Date:** Monday, 13th July 2026

---

### Task 1: HCL Syntax

### Anatomy of a Block

Every block in Terraform follows this structure:

```hcl
block_type "label_one" "label_two" {
  argument = value
}
```

Real example:
```hcl
resource "azurerm_resource_group" "my_rg" {
  name     = "my-rg"
  location = "East US"
}
```

Breaking it down:
- `resource` — block type (what kind of thing this is)
- `"azurerm_resource_group"` — label one (what type of resource)
- `"my_rg"` — label two (your local name to reference it)
- `name`, `location` — arguments (settings inside the block)

---

### Argument vs Block

| | Argument | Block |
|---|---|---|
| What it is | A key = value setting | A nested section with its own arguments |
| Example | `name = "my-rg"` | `validation { ... }` |
| Has curly braces? | No | Yes |

```hcl
variable "environment" {
  description = "Deployment env"   # ← argument
  type        = string             # ← argument

  validation {                     # ← nested block
    condition     = var.environment != ""
    error_message = "Cannot be empty."
  }
}
```

---

### Expressions

**String interpolation — embed values inside strings:**
```hcl
name = "my-${var.environment}-rg"
# if environment = "prod" → "my-prod-rg"
```

**References — point to another resource's value:**
```hcl
resource_group_name = azurerm_resource_group.my_rg.name
# reads the name from the resource group you created
```

**Operators:**
```hcl
count  = var.environment == "prod" ? 3 : 1   # conditional
length = var.count + 1                         # arithmetic
enable = var.debug && var.environment != "prod" # logical
```

---

## Task 2: Variables, Types & Validation

### variables.tf

```hcl
# ── Primitives ────────────────────────────────────────────

variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "environment must be one of: dev, staging, prod."
  }
}

variable "instance_count" {
  description = "Number of instances to create"
  type        = number
  default     = 1
}

variable "enable_monitoring" {
  description = "Enable monitoring for resources"
  type        = bool
  default     = true
}

# ── Collections ───────────────────────────────────────────

variable "allowed_regions" {
  description = "List of allowed Azure regions"
  type        = list(string)
  default     = ["East US", "West Europe", "Southeast Asia"]
}

variable "tags" {
  description = "Tags to apply to all resources"
  type        = map(string)
  default     = {
    project = "terraweek"
    owner   = "jeeni"
  }
}

variable "allowed_environments" {
  description = "Set of allowed environments (no duplicates)"
  type        = set(string)
  default     = ["dev", "staging", "prod"]
}

# ── Structural ────────────────────────────────────────────

variable "app_config" {
  description = "Application configuration object"
  type = object({
    name    = string
    port    = number
    enabled = bool
  })
  default = {
    name    = "myapp"
    port    = 8080
    enabled = true
  }
}

variable "app_tuple" {
  description = "Fixed structure: [name, port]"
  type        = tuple([string, number])
  default     = ["myapp", 8080]
}

# ── Sensitive ─────────────────────────────────────────────

variable "db_password" {
  description = "Database password — never shown in logs or plan output"
  type        = string
  sensitive   = true
}
```

---

### Variable types explained simply

| Type | What it stores | Example |
|---|---|---|
| `string` | Text | `"dev"` |
| `number` | Number | `3` |
| `bool` | True or false | `true` |
| `list(string)` | Ordered list, duplicates allowed | `["a", "b", "a"]` |
| `map(string)` | Key-value pairs | `{ env = "dev" }` |
| `set(string)` | Unordered list, no duplicates | `["a", "b"]` |
| `object({...})` | Group of named attributes | `{ name = "x", port = 80 }` |
| `tuple([...])` | Fixed list of mixed types | `["myapp", 8080]` |

---

## Task 3: Locals, Outputs & Functions

### locals.tf

```hcl
locals {
  # compute a reusable name prefix
  name_prefix = "${var.environment}-terraweek"

  # merge default tags with extra tags
  common_tags = merge(var.tags, {
    environment = var.environment
    managed_by  = "terraform"
  })

  # uppercase the environment for display
  env_upper = upper(var.environment)

  # join values into a single string
  resource_name = join("-", ["tws", var.environment, "2026"])
}
```

### outputs.tf

```hcl
output "name_prefix" {
  description = "Common name prefix used across resources"
  value       = local.name_prefix
}

output "common_tags" {
  description = "Tags applied to all resources"
  value       = local.common_tags
}

output "resource_name" {
  description = "Generated resource name"
  value       = local.resource_name
}

output "db_password" {
  description = "Database password"
  value       = var.db_password
  sensitive   = true   # won't show in terminal output
}
```

---

### Built-in Functions (tested in terraform console)

```bash
terraform console
```

```hcl
> upper("terraweek")
"TERRAWEEK"

> merge({a=1}, {b=2})
{ "a" = 1, "b" = 2 }

> join("-", ["tws", "terraweek", "2026"])
"tws-terraweek-2026"

> length(["dev", "staging", "prod"])
3

> format("Hello, %s!", "Jeeni")
"Hello, Jeeni!"

> lookup({env = "dev"}, "env", "unknown")
"dev"
```

---

## Task 4: Docker Provider — Hands On

### main.tf

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {}

variable "container_name" {
  type    = string
  default = "tws-web"
}

variable "external_port" {
  type    = number
  default = 8080
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "web" {
  name  = var.container_name
  image = docker_image.nginx.image_id

  ports {
    internal = 80
    external = var.external_port
  }
}

output "container_url" {
  value = "http://localhost:${var.external_port}"
}
```

### Commands run

```bash
terraform init
terraform plan  -var 'container_name=tws-web' -var 'external_port=8080'
terraform apply -var 'container_name=tws-web' -var 'external_port=8080'
# visit http://localhost:8080
terraform output
terraform destroy -var 'container_name=tws-web' -var 'external_port=8080'
```

### Using terraform.tfvars instead of -var flags

```hcl
# terraform.tfvars
container_name = "tws-web"
external_port  = 8080
```

Then just run:
```bash
terraform apply
# Terraform automatically picks up terraform.tfvars — no -var flags needed
```

**Difference:** `-var` flags are for one-off runs. `terraform.tfvars` is for values you always want set — cleaner and less typing.

---

## Variable Precedence (highest to lowest)

```
-var / -var-file flags        ← highest — overrides everything
       ↓
*.auto.tfvars                 ← auto-loaded tfvars files
       ↓
terraform.tfvars              ← default tfvars file
       ↓
TF_VAR_ environment variables ← set in shell before running
       ↓
default in variable block     ← lowest — used if nothing else set
```

---

## Bonus

### for expression — transform a list

```hcl
variable "names" {
  type    = list(string)
  default = ["dev", "staging", "prod"]
}

locals {
  upper_names = [for s in var.names : upper(s)]
  # result: ["DEV", "STAGING", "PROD"]
}
```

### Conditional expression

```hcl
locals {
  instance_type = var.environment == "prod" ? "t3.medium" : "t3.micro"
  # prod → t3.medium, anything else → t3.micro
}
```

### optional() inside object type

```hcl
variable "app_config" {
  type = object({
    name    = string
    port    = optional(number, 8080)   # optional — defaults to 8080 if not set
    enabled = optional(bool, true)     # optional — defaults to true if not set
  })
}
```

---

## Output from hands-on run

```bash
# paste your terraform init output here
# paste your terraform plan output here
# paste your terraform apply output here
# paste your terraform output here
# paste your terraform destroy output here
```

---

*Source: TerraWeek Challenge — Day 2*
