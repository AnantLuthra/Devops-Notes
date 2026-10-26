# Terraform Notes

## Table of Contents
- [Terraform Notes](#terraform-notes)
  - [Table of Contents](#table-of-contents)
  - [What is Terraform?](#what-is-terraform)
  - [Difference between Terraform and Ansible](#difference-between-terraform-and-ansible)
  - [Terraform Architecture](#terraform-architecture)
    - [3 levels on which terraform works](#3-levels-on-which-terraform-works)
  - [Declarative vs Imperative](#declarative-vs-imperative)
  - [Basic Terraform Commands for different stages](#basic-terraform-commands-for-different-stages)
  - [Different type of Providers \& installation](#different-type-of-providers--installation)
  - [Resource \& Data Sources](#resource--data-sources)
  - [Changing \& Destroying Resources](#changing--destroying-resources)
  - [Terraform Commands](#terraform-commands)
  - [Terraform State \& its commands](#terraform-state--its-commands)
    - [Commands related to Terraform State:](#commands-related-to-terraform-state)
  - [Output in Terraform](#output-in-terraform)
  - [Variables in Terraform](#variables-in-terraform)
    - [3 Ways to pass values of variables in terraform:](#3-ways-to-pass-values-of-variables-in-terraform)
    - [Usecases \& usage of variables in terraform:](#usecases--usage-of-variables-in-terraform)

## What is Terraform?
- Terraform is an open-source infrastructure as code (IaC) tool that allows you to define and provision infrastructure using a high-level configuration language. It enables you to manage cloud services, virtual machines, networks, and other resources in a consistent and repeatable manner.
- Example: Lets say we've an app where we first need certain ec2 instances with specific configurations, then for creating we use terraform, and we can use it for later adjusting, and having that same infrastructure in different environments like dev, staging, and production. It allows us to version control our infrastructure and easily replicate it across different environments.


## Difference between Terraform and Ansible
- Terraform is primarily focused on infrastructure provisioning and management, while Ansible is a configuration management tool that automates software deployment and configuration tasks.
- Ansible is reletively mature and has a large community, while Terraform is newer but rapidly gaining popularity.
- Terraform is much more advanced in orchestration.
- So Terraform is better tool for provisioning infrastructure, while Ansible is better suited for configuration management and application deployment.

## Terraform Architecture
- So it has 2 input sources.
  1. **Configuration files:** These are the files where you define your infrastructure using HashiCorp Configuration Language (HCL) or JSON. They describe the desired state of your infrastructure, including resources, variables, and outputs.
  2. **State files:** Terraform maintains a state file that keeps track of the current state of your infrastructure. It allows Terraform to determine what changes need to be made to reach the desired state defined in the configuration files.
- So through these 2 inputs it finds out the difference between the current state and the desired state, and then it generates an execution plan that outlines the actions required to achieve the desired state - which is very similar to how kubernetes works.
- 2nd part of the architecture is **Providers**
  - Providers are responsible for interacting with the underlying infrastructure platforms (e.g., AWS, Azure, GCP) and managing the resources defined in the configuration files. They provide the necessary APIs and functionality to create, update, and delete resources.
  - There are providers for various cloud platforms, aws, azure, even kubernetes - so that terraform can create resources in kubernetes as well.
- Because of these providers with terraform you have one tool to manage your infrastructure across multiple platforms, which is a huge advantage.

Below is the example of a configuration file:
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"
}
```

### 3 levels on which terraform works
- **Infrastructure as Code (IaC):** Like creating resources in AWS, Azure etc.
- **Platform as a Service (PaaS):** Like creating resources in kubernetes, like deployments, services etc.
- **Software as a Service (SaaS):** Like creating resources in SaaS platforms, like creating a new workspace in slack, or creating a new repo in github etc.

## Declarative vs Imperative
- Declarative: In this we just mention a final state of our infrastructure, and terraform will figure out how to reach that state. It is more like a "what" approach.
- Imperative: In this we mention the steps to reach a final state, and terraform will follow those steps. It is more like a "how" approach. For example, we'll ask it to remove a instance, create a permission etc.

Benefits of declarative approach:
- Adjust old config file and re-execute
- Easy to maintain and update
- Clean and small config file
- Always know the current setup

## Basic Terraform Commands for different stages
- refersh: `terraform refresh` - This command is used to update the state file with the current state of the infrastructure. It checks the actual resources in the cloud provider and updates the state file accordingly.
- plan: `terraform plan` - This command is used to create an execution plan that shows the changes Terraform will make to reach the desired state defined in the configuration files. It allows you to review the proposed changes before applying them.
- apply: `terraform apply` - This command is used to apply the changes defined in the execution plan and create or modify the infrastructure resources. It provisions the resources based on the configuration files and updates the state file accordingly.
- destroy: `terraform destroy` - This command is used to destroy the infrastructure resources managed by Terraform. It removes all the resources defined in the configuration files and updates the state file accordingly.


## Different type of Providers & installation
1. **HashiCorp Providers:** These are the official providers developed and maintained by HashiCorp. They cover a wide range of cloud platforms, including AWS, Azure, GCP, and more. You can install them using the `terraform init` command, which automatically downloads the required providers based on the configuration files.
2. **Partner Providers:** These are providers developed by HashiCorp's partners. They may offer additional functionality or support for specific platforms. You can install them using the `terraform init` command, similar to HashiCorp providers.
3. **Community Providers:** These are providers developed by the Terraform community. They may cover niche platforms or provide specialized functionality. You can install them by specifying the provider source in the configuration files and running `terraform init`.
4. **Individual Providers:** These are providers developed by individual developers or organizations. They may offer unique features or support for specific use cases. You can install them by specifying the provider source in the configuration files and running `terraform init`.

- In case of all it is a must to create seperate `providers.tf` file, where we mention the provider source and version, and then run `terraform init` to install the provider.
Example of a `providers.tf` file:
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
    linode = {
      source  = "linode/linode"
      version = "4.8.0"
    }
  }
}
```

## Resource & Data Sources
- We create resources in terraform using `resource` block, and we can use `data` block to get information about existing resources. For example, if we want to create a new ec2 instance, we can use `resource` block, and if we want to get information about an existing vpc, we can use `data` block.

Example of a `resource` block:
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

Example of a `data` block:
```hcl
data "aws_vpc" "example" {
  id = "vpc-12345678"
}
```

## Changing & Destroying Resources
- To change a resource, we can simply modify the configuration file and run `terraform apply`. Terraform will detect the changes and update the resource accordingly. For example, if we want to change the instance type of an existing EC2 instance, we can modify the `instance_type` attribute in the `resource` block and run `terraform apply`.

- To destroy a resource, we have 3 options:
  1. We can use `terraform destroy` command, which will destroy all the resources defined in the configuration files.
  2. To destroy only the specified resource - We can use `terraform destroy -target=<resource_name>` command, For example, if we want to destroy only the EC2 instance, we can run `terraform destroy -target=aws_instance.example`.
  3. We can edit the configuration file and remove the resource block which we wanna destroy, and then run `terraform apply`. Terraform will detect that the resource is no longer defined in the configuration files and will destroy it accordingly.

Example of point 2 for destroying a specific resource:
```bash
terraform destroy -target=aws_subnet.dev-subnet-2
```
- But we should always change configuration file and remove the resource block which we wanna destroy, and then run `terraform apply`. This is the best practice to follow, because it will keep our configuration files clean and up-to-date with the current state of our infrastructure.

## Terraform Commands
- `terraform plan`: This will create an execution plan that shows the changes Terraform will make to reach the desired state defined in the configuration files. **It shows all attributes of the resources as well.** It allows you to review the proposed changes before applying them.
- `terraform apply -auto-approve`: This command is used to apply the changes defined in the execution plan and create or modify the infrastructure resources. It provisions the resources based on the configuration files and updates the state file accordingly. The `-auto-approve` flag automatically approves the changes without prompting for confirmation.
- `terraform destroy`: This command is used to destroy the infrastructure resources managed by Terraform. It removes all the resources defined in the configuration files in the correct order.

## Terraform State & its commands
- Terraform maintains a `terraform.tfstate` file that keeps track of the current state of your infrastructure. It allows Terraform to determine what changes need to be made to reach the desired state defined in the configuration files. 
- Terraform also maintains a `terraform.tfstate.backup` file that serves as a backup of the state file. It is automatically created whenever the state file is updated, providing a safety net in case of accidental changes or corruption.

### Commands related to Terraform State:
- `terraform state list`: This command is used to list all the resources currently managed by Terraform in the state file. It provides a summary of the resources and their current state.
- `terraform state show <resource_name>`: This command is used to display detailed information about a specific resource in the state file. It shows the resource's attributes, dependencies, and current state.
- `terraform state rm <resource_name>`: This command is used to remove a specific resource from the state file. It allows you to manually remove resources that are no longer managed by Terraform or have been deleted outside of Terraform.

## Output in Terraform
- We have `output` block in terraform, which is used to define output values that can be displayed after the execution of `terraform apply`. It allows you to retrieve and display important information about the created resources, such as IP addresses, URLs, or any other relevant data.

Example of an `output` block:
```hcl

output "output1"{
    value = aws_vpc.my_vpc.arn
}

output "output2"{
    value = aws_subnet.subnet_1.arn
}
```

## Variables in Terraform
- Variables in Terraform are used to parameterize the configuration files, allowing you to define values that can be easily changed without modifying the code. They provide flexibility and reusability in your infrastructure definitions.

Example of a `variable` block:
```hcl
variable "vpc_cidr_block"{
    description = "cidr block value for vpc."
    type = string
    default = "10.0.0.0/16"
}
```
> Using `default` above will make the variable to be passed through cli or through `terraform.tfvars` file optional, and if not passed then the default value will be used.
> Using `type` above will make the variable to be passed through cli or through `terraform.tfvars` file mandatory, and if not passed then terraform will throw an error.
> 

### 3 Ways to pass values of variables in terraform:
1. Manually entering values during `terraform apply` execution.
2. Through CLI - Example - `terraform apply -var="vpc_cidr_block=10.0.0.0/16"`
3. Using `terraform.tfvars` file - Example:
```hcl
vpc_cidr_block = "10.0.0.0/16"
```

### Usecases & usage of variables in terraform:
- To make the configuration more flexible and reusable for different environments (e.g., dev, staging, production) by allowing you to define different values for the same variable in each environment.
- Default `terraform.tfvars` file is automatically used by terraform, other than that we'll have to provide the file name using `-var-file` flag, like `terraform apply -var-file="dev.tfvars"`. 
- This allows you to have multiple variable files for different environments and easily switch between them during deployment.
