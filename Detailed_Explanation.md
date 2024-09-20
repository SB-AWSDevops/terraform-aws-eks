 This repository contains Terraform code to provision an Amazon EKS (Elastic Kubernetes Service) cluster on AWS. 

---

### Table of Contents

1. **Introduction**
   - What is Terraform?
   - What is Amazon EKS?
2. **Repository Structure**
   - Overview of files and directories
3. **Detailed Explanation**
   - `main.tf`
   - `variables.tf`
   - `outputs.tf`
   - `provider.tf`
   - `versions.tf`
   - `modules/` directory
   - Other supporting files
4. **How to Use the Repository**
   - Prerequisites
   - Steps to deploy the EKS cluster
5. **Best Practices and Considerations**
6. **Conclusion**

---

## 1. Introduction

### What is Terraform?

Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It allows you to define and provision data center infrastructure using a declarative configuration language known as HashiCorp Configuration Language (HCL).

### What is Amazon EKS?

Amazon Elastic Kubernetes Service (EKS) is a managed service that simplifies running Kubernetes on AWS without the need to install, operate, and maintain your own Kubernetes control plane or nodes.

---

## 2. Repository Structure

The repository is organized to facilitate the deployment of an EKS cluster using Terraform. Here's an overview of the main files and directories:

- **`main.tf`**: The primary Terraform configuration file where resources are defined.
- **`variables.tf`**: Contains variable declarations used throughout the Terraform code.
- **`outputs.tf`**: Defines output values that are exported after deployment.
- **`provider.tf`**: Configures the AWS provider.
- **`versions.tf`**: Specifies required Terraform and provider versions.
- **`modules/`**: A directory containing reusable Terraform modules.
- **`README.md`**: Documentation for the repository.
- **Other supporting files**: Such as `locals.tf`, `data.tf`, etc.

---

## 3. Detailed Explanation

Let's delve into each file and explain its purpose and content.

### `main.tf`

This is the main configuration file where the core infrastructure is defined.

**Key Components:**

1. **VPC Creation**: Defines the Virtual Private Cloud where the EKS cluster will reside.

   ```hcl
   resource "aws_vpc" "eks_vpc" {
     cidr_block = var.vpc_cidr
     ...
   }
   ```

2. **Subnets**: Creates public and private subnets across multiple availability zones.

   ```hcl
   resource "aws_subnet" "public_subnets" {
     count             = length(var.public_subnet_cidrs)
     vpc_id            = aws_vpc.eks_vpc.id
     cidr_block        = var.public_subnet_cidrs[count.index]
     availability_zone = data.aws_availability_zones.available.names[count.index]
     ...
   }
   ```

3. **Internet Gateway and NAT Gateway**: Allows instances in public subnets to reach the internet.

   ```hcl
   resource "aws_internet_gateway" "igw" {
     vpc_id = aws_vpc.eks_vpc.id
     ...
   }

   resource "aws_nat_gateway" "nat" {
     allocation_id = aws_eip.nat_eip.id
     subnet_id     = aws_subnet.public_subnets[0].id
     ...
   }
   ```

4. **Route Tables**: Manages routing between subnets and gateways.

   ```hcl
   resource "aws_route_table" "public_rt" {
     vpc_id = aws_vpc.eks_vpc.id
     ...
   }
   ```

5. **Security Groups**: Defines security groups for the EKS cluster and worker nodes.

   ```hcl
   resource "aws_security_group" "eks_cluster_sg" {
     name        = "${var.cluster_name}-eks-cluster-sg"
     description = "Security group for EKS cluster"
     vpc_id      = aws_vpc.eks_vpc.id
     ...
   }
   ```

6. **EKS Cluster**: Creates the EKS cluster with the control plane.

   ```hcl
   resource "aws_eks_cluster" "eks_cluster" {
     name     = var.cluster_name
     role_arn = aws_iam_role.eks_cluster_role.arn
     ...
   }
   ```

7. **Node Groups**: Provisions worker nodes for the EKS cluster.

   ```hcl
   resource "aws_eks_node_group" "node_group" {
     cluster_name    = aws_eks_cluster.eks_cluster.name
     node_group_name = "${var.cluster_name}-node-group"
     ...
   }
   ```

### `variables.tf`

Defines all the variables used in `main.tf` and other configuration files. This allows for customization without altering the core code.

**Examples:**

- **`aws_region`**: The AWS region to deploy resources.
- **`cluster_name`**: The name of the EKS cluster.
- **`vpc_cidr`**: The CIDR block for the VPC.
- **`public_subnet_cidrs`**: List of CIDR blocks for public subnets.
- **`private_subnet_cidrs`**: List of CIDR blocks for private subnets.

```hcl
variable "aws_region" {
  description = "The AWS region to deploy resources in."
  type        = string
  default     = "us-east-1"
}
```

### `outputs.tf`

Specifies the outputs that will be displayed after `terraform apply` is run. These outputs can be used by other Terraform configurations or for quick reference.

**Examples:**

- **`cluster_id`**: The ID of the EKS cluster.
- **`cluster_endpoint`**: The endpoint URL for the Kubernetes API server.
- **`cluster_certificate_authority_data`**: The certificate data for cluster authentication.

```hcl
output "cluster_id" {
  description = "The ID of the EKS cluster."
  value       = aws_eks_cluster.eks_cluster.id
}
```

### `provider.tf`

Configures the AWS provider, including the region and any additional settings.

```hcl
provider "aws" {
  region = var.aws_region
}
```

### `versions.tf`

Ensures that the correct versions of Terraform and the AWS provider are used, which helps maintain compatibility and prevent unexpected issues.

```hcl
terraform {
  required_version = ">= 0.13"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}
```

### `modules/` Directory

Contains reusable modules that encapsulate specific functionality, promoting code reuse and modularity.

**Possible Modules:**

- **`vpc/`**: Manages VPC creation.
- **`eks/`**: Handles EKS cluster provisioning.
- **`node_group/`**: Manages EKS worker nodes.

**Example Usage:**

In `main.tf`, you might see:

```hcl
module "vpc" {
  source = "./modules/vpc"
  ...
}
```

### Other Supporting Files

- **`data.tf`**: Contains data sources that fetch existing information from AWS.
- **`locals.tf`**: Defines local values that can simplify expressions.
- **`README.md`**: Provides documentation on how to use the repository.

---

## 4. How to Use the Repository

### Prerequisites

- **Terraform Installed**: Ensure you have Terraform version `>= 0.13`.
- **AWS Account**: An AWS account with permissions to create resources.
- **AWS CLI Configured**: Set up your AWS credentials locally.

### Steps to Deploy the EKS Cluster

1. **Clone the Repository**

   ```bash
   git clone https://github.com/SB-AWSDevops/terraform-aws-eks.git
   cd terraform-aws-eks/main
   ```

2. **Customize Variables**

   You can either edit `variables.tf` directly or create a `terraform.tfvars` file.

   **Example `terraform.tfvars`:**

   ```hcl
   aws_region          = "us-east-1"
   cluster_name        = "my-eks-cluster"
   vpc_cidr            = "10.0.0.0/16"
   public_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
   private_subnet_cidrs = ["10.0.3.0/24", "10.0.4.0/24"]
   ```

3. **Initialize Terraform**

   ```bash
   terraform init
   ```

4. **Plan the Deployment**

   Review the resources that will be created.

   ```bash
   terraform plan
   ```

5. **Apply the Configuration**

   Deploy the EKS cluster.

   ```bash
   terraform apply
   ```

   Confirm the action when prompted.

6. **Configure `kubectl`**

   Set up your Kubernetes configuration to interact with the cluster.

   ```bash
   aws eks --region us-east-1 update-kubeconfig --name my-eks-cluster
   ```

7. **Verify the Cluster**

   Use `kubectl` to verify that the cluster is running.

   ```bash
   kubectl get nodes
   ```

8. **Destroy the Resources**

   When you're done, clean up to avoid unnecessary charges.

   ```bash
   terraform destroy
   ```

---

## 5. Best Practices and Considerations

- **Version Control**: Keep your Terraform code in a version control system like Git.
- **State Management**: Use remote state backends (e.g., S3 with DynamoDB locking) for collaborative environments.
- **Security**: Limit access to your AWS credentials and use IAM roles with least privilege.
- **Modularity**: Use modules to encapsulate and reuse code.
- **Variable Management**: Use `terraform.tfvars` or environment variables to manage sensitive information.
- **Resource Naming**: Use consistent naming conventions for resources.
- **Monitoring and Logging**: Set up CloudWatch and other monitoring tools to keep an eye on your cluster.
- **Cost Management**: Be aware of the costs associated with running EKS clusters and manage resources accordingly.

---

## 6. Conclusion

This repository provides a Terraform-based solution to deploy an Amazon EKS cluster on AWS. By understanding each component and following the best practices, you can effectively manage and scale your Kubernetes workloads in the cloud.

---

**Note**: Always make sure to review and understand the code before deploying it, especially in a production environment. The configurations should be adjusted to fit your specific needs and compliance requirements.
