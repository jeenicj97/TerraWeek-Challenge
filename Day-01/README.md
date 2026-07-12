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
     

- How can you install Terraform and set up the environment for AWS, Azure, or GCP?

- Explain the important terminologies of Terraform with the example at least (5 crucial terminologies).
