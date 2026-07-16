# TerraWeek Day 4 — State & Remote Backends

**Date:** Wednesday, 15th July 2026

---

## What Changed in 2026 — Important!

Old way → S3 + DynamoDB for state locking
New way → S3 native locking with `use_lockfile = true` (Terraform 1.11+)

**DynamoDB-based locking is now deprecated — don't use it for new projects.**

---

## Task 1: Why State Matters

### What is terraform.tfstate?

The state file is Terraform's memory — it records everything Terraform created and manages.

It stores:
- Every resource Terraform created (EC2, VPC, S3 buckets etc)
- The real cloud IDs of those resources (e.g. `i-0abc123` for an EC2)
- Attribute values — IPs, ARNs, names, sizes
- Dependencies between resources

Without the state file, Terraform has no idea what it already created — it would try to create everything again from scratch.

---

### Why never edit it by hand or commit to Git?

**Never edit by hand:**
- It's JSON with precise formatting — one wrong character breaks everything
- Terraform will lose track of your infrastructure
- Use `terraform state` commands instead — they edit safely

**Never commit to Git:**
- State files contain **plaintext secrets** — database passwords, access keys, private IPs
- Anyone with repo access can read them
- Use a remote backend (S3, Azure Storage) instead — with encryption

---

### What is State Drift?

State drift happens when your real cloud infrastructure is different from what Terraform's state file says.

**Example:**
```
Terraform state says:  EC2 instance type = t3.micro
Someone manually changed it in AWS console to: t3.small
Now they don't match — this is drift
```

**How to detect and fix:**
```bash
terraform plan      # compares state vs real infra, shows differences
terraform refresh   # updates state to match real infra (use carefully)
```

---

### Why is state sensitive?

State files store real values in plaintext — things like:
- Database passwords
- Private IP addresses
- Access keys and secrets
- Connection strings

This is why state must be:
- Stored remotely with **encryption at rest**
- Access controlled — not everyone should read it
- Never committed to Git

---

## Task 2: terraform state commands

```bash
# list all resources Terraform is managing
terraform state list

# inspect one specific resource in detail
terraform state show aws_instance.web

# rename a resource in state (without destroying it)
terraform state mv aws_instance.web aws_instance.webserver

# stop managing a resource (does NOT delete the actual infra)
terraform state rm aws_instance.web

# human-readable view of entire state
terraform show
```

### When to use each:

| Command | When to use |
|---|---|
| `state list` | Get an overview of everything Terraform manages |
| `state show` | Debug a specific resource — see its actual values |
| `state mv` | Rename a resource in code without destroying it |
| `state rm` | Stop managing a resource but keep it running in cloud |
| `show` | Full readable dump — useful during troubleshooting |

---

## Task 3: Bootstrap the Backend (S3 Bucket)

The S3 bucket that stores your state must exist **before** you configure the backend.
Create it first using a separate config with local state:

```hcl
# backend_infra/main.tf
resource "aws_s3_bucket" "tfstate" {
  bucket = "your-unique-terraweek-state-bucket"

  tags = {
    Name    = "Terraform State Bucket"
    Project = "terraweek"
  }
}

# enable versioning — lets you recover old state versions
resource "aws_s3_bucket_versioning" "tfstate" {
  bucket = aws_s3_bucket.tfstate.id

  versioning_configuration {
    status = "Enabled"
  }
}

# enable encryption — state contains secrets
resource "aws_s3_bucket_server_side_encryption_configuration" "tfstate" {
  bucket = aws_s3_bucket.tfstate.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# block all public access — state should never be public
resource "aws_s3_bucket_public_access_block" "tfstate" {
  bucket                  = aws_s3_bucket.tfstate.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

```bash
cd backend_infra
terraform init
terraform apply    # creates the S3 bucket with local state
```

---

## Task 4: Configure Remote Backend with Native Locking

Now point your actual config at that S3 bucket:

```hcl
# backend_demo/terraform.tf
terraform {
  required_version = ">= 1.11"

  backend "s3" {
    bucket       = "your-unique-terraweek-state-bucket"
    key          = "day04/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    use_lockfile = true    # native S3 locking — no DynamoDB needed!
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
```

```bash
cd backend_demo
terraform init     # Terraform offers to migrate local state → S3
terraform apply
```

### What happens during apply with locking:

```
terraform apply starts
    ↓
.tflock file appears in S3 bucket  ← someone else can't run apply now
    ↓
resources are created/updated
    ↓
.tflock file disappears            ← lock released, others can run now
    ↓
terraform.tfstate updated in S3
```

### Why locking matters:

Without locking — two people run `terraform apply` at the same time:
- Both read the same state
- Both make changes
- One overwrites the other's state
- Infrastructure becomes inconsistent

With locking — second person gets:
```
Error: Error acquiring the state lock
Another operation is currently running
```
They must wait — state stays consistent.

---

## Task 5: Import Existing Resources

Someone created an S3 bucket manually in the AWS console. Now you want Terraform to manage it.

**Step 1 — add an import block:**
```hcl
# import.tf
import {
  to = aws_s3_bucket.imported
  id = "my-manually-created-bucket"
}
```

**Step 2 — generate config automatically:**
```bash
terraform plan -generate-config-out=generated.tf
```

Terraform reads the real bucket and writes the Terraform config for it into `generated.tf`.

**Step 3 — review generated.tf, then apply:**
```bash
terraform apply
```

The bucket is now under Terraform management — no manual clicking needed again.

---

## Bonus

### Remote Backend Comparison

| Backend | Best for | Locking |
|---|---|---|
| S3 | AWS teams | Native `use_lockfile = true` |
| Azure Storage | Azure teams | Native blob leasing |
| GCS | GCP teams | Native object locking |
| HCP Terraform | Any team, managed service | Built-in, no setup needed |

---

### moved block — rename without destroying

```hcl
# renamed aws_instance.web → aws_instance.webserver in code
moved {
  from = aws_instance.web
  to   = aws_instance.webserver
}
```
Terraform updates state only — no destroy and recreate.

---

### removed block — stop managing without deleting

```hcl
removed {
  from = aws_s3_bucket.old_bucket

  lifecycle {
    destroy = false    # keep it in AWS, just stop managing it
  }
}
```

---

### check block — continuous health assertion

```hcl
check "bucket_is_private" {
  assert {
    condition     = aws_s3_bucket.tfstate.bucket_policy != ""
    error_message = "State bucket must have a bucket policy!"
  }
}
```
Runs on every plan and apply — like a health check for your infrastructure.

---

## Cleanup — always do this!

```bash
cd backend_demo && terraform destroy
cd ../backend_infra && terraform destroy
# empty the S3 bucket first if versioning is enabled
```

---

## Output from hands-on run

```bash
# terraform state list
# paste output here

# terraform state show <resource>
# paste output here

# terraform apply (backend_demo)
# paste output here
```

---

## Key Takeaways

- State = Terraform's memory. Without it, Terraform is blind.
- Never commit state to Git — it contains secrets in plaintext
- Always use remote backend with encryption for team work
- Use `use_lockfile = true` — no DynamoDB needed anymore
- `terraform state` commands are your tools for fixing state problems safely

---

*Source: TerraWeek Challenge — Day 4*
