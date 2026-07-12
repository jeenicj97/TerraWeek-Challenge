## TerraWeek Day 1

### Day 1: Introduction to Terraform and Terraform Basics

1. What is Terraform and how can it help you manage infrastructure as code?
   
    * Terraform is an open-source **Infrastructure as Code (IaC)** tool by HashiCorp that lets you define, provision and manage infrastructure using declarative configuration files (written in HCL - HashiCorp Configuration Language) instead of clicking through cloud consoles or writing imperative scripts.
       
    * Say, for example, instead of manually creating resources (like EC2, VM, Kubernetes) through the Azure/AWS portal, you describe the desired state in a `.tf` file, and Terraform figures out the steps needed to get there.   

2. Why do we need Terraform, and how does it simplify infrastructure provisioning?

   * Without Terraform, teams often rely on manual clicks or ad-hoc scripts, which are error-prone and hard to scale. Terraform simplifies this by:

        * Automating provisioning across multiple cloud providers
          
        * Tracking resource states to detect drift
          
        * Enabling collaboration through versioned configuration files
          
        * Supporting modular, reusable infrastructure components.

    * In short, Terraform transforms infrastructure managemnet from manual operations to automated, auditable workflows.
     

3. How can you install Terraform and set up the environment for AWS, Azure, or GCP?

      ### Installing Terraform (Linux/WSL2 — Ubuntu)
       
      ```bash
      # Install dependencies
      sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
       
      # Add HashiCorp GPG key
      curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
       
      # Add HashiCorp repo
      echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
       
      # Install
      sudo apt-get update && sudo apt-get install terraform
       
      # Verify
      terraform -version
      ```
       
      ### Setting up provider authentication
       
      **Azure:**
      ```bash
      # Install Azure CLI, then log in
      az login
       
      # Terraform's azurerm provider picks up credentials automatically
      # via the Azure CLI session
      ```
       
      ```hcl
      terraform {
        required_providers {
          azurerm = {
            source  = "hashicorp/azurerm"
            version = "~> 3.0"
          }
        }
      }
       
      provider "azurerm" {
        features {}
      }
      ```
       
      **AWS:**
      ```bash
      # Install AWS CLI, then configure credentials
      aws configure
      ```
      ```hcl
      provider "aws" {
        region = "us-east-1"
      }
      ```
       
      **GCP:**
      ```bash
      # Install gcloud CLI, then authenticate
      gcloud auth application-default login
      ```
      ```hcl
      provider "google" {
        project = "my-project-id"
        region  = "us-central1"
      }
      ```
       
      Once the provider block is set and credentials are in place, running `terraform init` downloads the required provider plugins into a local `.terraform` directory.

4. Explain the important terminologies of Terraform with the example at least (5 crucial terminologies).

      1. **Provider** A plugin that lets Terraform interact with a specific platform's API (Azure, AWS, GCP, Kubernetes, etc.)

         ```hcl
         provider "azurerm" {
            features{}
         }
         ```
      2. **Resource** The building block - represents a single infrastructure object Terraform manages (a VM, storage account, resource group, etc.)

         ```hcl
         resource "azurerm_resource_group" "rg" {
            name = "terraweek-rg"
            location = "East US"
         }
      3. **State** A JSON file(`terraform.tfstate`) that maps your configuration to real-world resources. Terraform uses this to know what already exists and what needs to change.
      4. **Variables** Parameterize your configuration so it isn't hardcoded, making it resuable across environments.

          ```hcl
          variable "location" {
             descritption = "Azure region"
             type = string
             default = "East US"
          }

          resource "azurerm_resource_group" "rg" {
             name = "terraweek-rg"
             location = var.location
          }
      5. **Plan/Apply/Destroy (the core workflow)**
            * `terraform `
         
