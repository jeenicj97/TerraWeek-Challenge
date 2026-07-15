# TerraWeek Day 3 — Providers, Resources & Your First Cloud Infra

**Date:** Tuesday, 14th July 2026

---

## 60-Second Networking Primer

Before touching code — understand what you're building:

| Block | What it is | Analogy |
|---|---|---|
| VPC | Your own private isolated network in the cloud | Your own gated neighbourhood |
| Subnet | A slice of the VPC's IPs, lives in one zone | A street in that neighbourhood |
| Internet Gateway | The door between your VPC and the public internet | The neighbourhood's main gate |
| Route Table | Rules that say "internet traffic → go via IGW" | Road signs / GPS routes |
| Security Group | Virtual firewall — which ports are open | A bouncer checking who gets in |
| EC2 Instance | The actual virtual machine running your app | A house on the street |

**How they connect:**
```
Internet ──▶ [IGW] ──▶ [Route Table] ──▶ [ Public Subnet ] ──▶ [SG] ──▶ [EC2]
                                          (inside the VPC)
```

---

## Task 1: Providers & Version Pinning

### What is version pinning?

Telling Terraform to only use a specific range of provider versions — so your code doesn't break when a new provider version is released with breaking changes.

```hcl
terraform {
  required_version = ">= 1.5.0"   # minimum Terraform version

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"           # pin to 6.x only
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"           # pin to 4.x only
    }
  }
}
```

### What does `~>` mean?

The `~>` operator is called the **pessimistic constraint operator**.

```
~> 6.0   means: >= 6.0 AND < 7.0   (allows 6.1, 6.2 but not 7.0)
~> 6.1.2 means: >= 6.1.2 AND < 6.2 (allows 6.1.3 but not 6.2.0)
```

Why it matters — without pinning:
- Today: provider v6.0 works perfectly
- Next month: provider v7.0 releases with breaking changes
- Your `terraform init` downloads v7.0 and everything breaks

With `~> 6.0` — Terraform stays on 6.x no matter what.

### Bonus: Provider Aliases (multiple regions)

```hcl
provider "aws" {
  region = "us-east-1"      # default provider
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"      # second provider with alias
}

# use the aliased provider for specific resources
resource "aws_instance" "west_server" {
  provider = aws.west       # uses us-west-2
  ami      = "ami-xxx"
  instance_type = "t3.micro"
}
```

**When to use aliases:**
- Deploying resources across multiple regions
- Disaster recovery setups (primary + backup region)
- Compliance requirements (data must stay in a specific region)

---

## Task 2: Resources vs Data Sources

### The difference

| | Resource | Data Source |
|---|---|---|
| What it does | Creates and manages something new | Reads existing information |
| Keyword | `resource` | `data` |
| Makes changes? | Yes — creates, updates, destroys | No — read only |
| Example | Create a new EC2 instance | Find the latest Amazon Linux AMI |

### Resource — creates something new

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "my-vpc"
  }
}
```

### Data Source — reads existing info

```hcl
# find the latest Amazon Linux 2023 AMI — don't hardcode AMI IDs!
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

# then reference it in your resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id   # uses the data source result
  instance_type = "t3.micro"
}
```

**Why use data sources?**
AMI IDs change every time Amazon releases a new version. If you hardcode `ami-0abc123`, it may not exist in 6 months. The data source always finds the latest one automatically.

---

## Task 3: Cloud Stack — AWS Example

### Provider setup (never hardcode credentials!)

```bash
# authenticate first
aws configure
# enter: Access Key ID, Secret Access Key, Region
```

```hcl
# providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

### Full stack — main.tf

```hcl
# ── Data Source: find latest Amazon Linux 2023 AMI ────────
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

# ── VPC ───────────────────────────────────────────────────
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "terraweek-vpc" }
}

# ── Subnet ────────────────────────────────────────────────
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  tags = { Name = "terraweek-subnet" }
}

# ── Internet Gateway ──────────────────────────────────────
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
  tags = { Name = "terraweek-igw" }
}

# ── Route Table ───────────────────────────────────────────
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"          # all internet traffic
    gateway_id = aws_internet_gateway.igw.id  # goes via IGW
  }

  tags = { Name = "terraweek-rt" }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# ── Security Group ────────────────────────────────────────
resource "aws_security_group" "web" {
  vpc_id = aws_vpc.main.id
  name   = "terraweek-sg"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]   # allow HTTP from anywhere
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]   # allow SSH (restrict in production!)
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]   # allow all outbound
  }

  tags = { Name = "terraweek-sg" }
}

# ── EC2 Instance ──────────────────────────────────────────
resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]

  tags = { Name = "terraweek-ec2" }
}
```

### Commands run

```bash
terraform init
terraform validate
terraform plan
terraform apply
terraform state list    # see everything Terraform manages
terraform destroy       # always clean up to avoid bills!
```

### terraform state list output

```
# paste your output here
aws_instance.web
aws_internet_gateway.igw
aws_route_table.public
aws_route_table_association.public
aws_security_group.web
aws_subnet.public
aws_vpc.main
data.aws_ami.amazon_linux
```

---

## Task 4: Meta-Arguments

### count — create N identical resources

```hcl
resource "aws_instance" "web" {
  count         = 3                          # creates 3 instances
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  tags = {
    Name = "web-${count.index}"             # web-0, web-1, web-2
  }
}
```

Access individual instances: `aws_instance.web[0]`, `aws_instance.web[1]`

---

### for_each — create from a map (preferred over count)

```hcl
variable "servers" {
  type = map(string)
  default = {
    web = "t3.micro"
    api = "t3.small"
    db  = "t3.medium"
  }
}

resource "aws_instance" "servers" {
  for_each      = var.servers
  ami           = data.aws_ami.amazon_linux.id
  instance_type = each.value                 # t3.micro, t3.small etc

  tags = {
    Name = each.key                          # web, api, db
  }
}
```

Access: `aws_instance.servers["web"]`, `aws_instance.servers["api"]`

---

### count vs for_each — which one?

| | count | for_each |
|---|---|---|
| Use when | N identical resources | Each resource has a unique name/identity |
| Delete middle one | Reindexes everything — dangerous | Only deletes that one — safe |
| Reference | `resource[0]`, `resource[1]` | `resource["name"]` |
| **Prefer** | Simple cases only | Almost always use this |

---

### depends_on — force explicit ordering

Terraform usually figures out ordering automatically from references. Use `depends_on` only when there's a hidden dependency it can't detect:

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  depends_on = [aws_internet_gateway.igw]   # wait for IGW before creating instance
}
```

---

### lifecycle — control how resources are managed

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  lifecycle {
    create_before_destroy = true
    # creates replacement BEFORE destroying old one
    # zero downtime replacements

    prevent_destroy = true
    # Terraform will refuse to destroy this resource
    # great for production databases

    ignore_changes = [tags["LastModified"]]
    # ignore changes to this specific tag
    # useful when something else (like AWS) auto-updates tags
  }
}
```

---

## Task 5: Update & Destroy

### Reading terraform plan output

```
~ aws_instance.web will be updated in-place   # ← safe, no downtime
  ~ tags = {
      + "Env" = "prod"
    }

-/+ aws_instance.web must be replaced         # ← destroys and recreates!
  ~ ami = "ami-old" → "ami-new"               # AMI changes force replace
```

**In-place update** — resource stays running, just config changes
**Forces replace** — resource is destroyed and recreated — causes downtime!

Things that typically force replace: AMI ID, instance type, subnet, VPC

### Always destroy when done

```bash
terraform destroy   # type: yes
# avoids surprise AWS bills!
```

---

## Bonus

### Elastic IP

```hcl
resource "aws_eip" "web" {
  instance = aws_instance.web.id
  domain   = "vpc"
}

output "public_ip" {
  value = aws_eip.web.public_ip
}
```

### User data — install Nginx on boot

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y nginx
    systemctl start nginx
    systemctl enable nginx
  EOF
}
```

### moved block — rename without destroying

```hcl
# renamed aws_instance.web → aws_instance.webserver
moved {
  from = aws_instance.web
  to   = aws_instance.webserver
}
# Terraform updates state without destroying and recreating
```

---

## Output from hands-on run

```bash
# terraform init
# paste output here

# terraform plan
# paste output here

# terraform apply
# paste output here

# terraform state list
# paste output here

# terraform destroy
# paste output here
```

---

*Source: TerraWeek Challenge — Day 3*
