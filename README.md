# Socks Shop Microservices Deployment

## Project overview:

The goal of this project is to deploy a microservices-based application using Infrastructure as Code (IaaC) on Kubernetes. This README provides a step-by-step guide to automate the deployment process, ensuring clarity, maintenance, and security.

# Prerequites

. Terraform installed

. AWS cli

. Jenkins installed and configured with github account

# Setting up the EKS cluster on AWS

## Defining provider;

![provider](/pictures/providers.png)
In the code above, the required version block sets the required version of Terraform to be at least version 1.0.
It declares that this configuration uses the AWS provider from HashiCorp's official provider registry.

The provider block indicates that we're using AWS. region is a variable that specifies the AWS region to use.this variable is defined elsewhere in your configuration and we're calling the variable using the "VAR" function.

## Setting variables

![variables](/pictures/variables.png)

### aws_region Variable:

This variable is used to define the AWS region where your resources will be created.
It has a description to provide information about its purpose.
The type attribute specifies that the variable is expected to be of type string.

### environment Variable:

This variable is intended to store an environment variable used as a prefix for resources.
It also has a description for clarity.
Like the aws_region variable, it's of type string.

### business_division Variable:

This variable is meant to represent the business division to which the infrastructure belongs.
It has a description for documentation purposes.
Similar to the other variables, it's of type string.

## Setting local values

![local](/pictures/local.png)

The Terraform code provided defines local values using the locals block. These local values are calculated based on the input variables and are meant to simplify the usage of values throughout your Terraform configuration.

For example, instead of referencing var.business_division in multiple places, you can use local.owners throughout your configuration. The same applies to other calculated local values like local.name and local.eks_cluster_name.

## Setting up VPC

The first step is creating a variables.tf file to set variables for your vpc like the one below;

![vpc](/pictures/vpc-variables.png)

### vpc modules;

![module](/pictures/module1.png)
![module1](/pictures/module2.png)

The Terraform code sets up an Amazon Virtual Private Cloud (VPC) using a Terraform module from the terraform-aws-modules/vpc/aws repository.

### AWS Availability Zones Datasource;

This block uses the aws_availability_zones data source to fetch information about the availability zones in the specified region. It excludes the availability zone "eu-west-2-iah-1a" from the list of available zones.

### VPC Terraform Module

#### VPC Basic Details:

name: Combines the business_division and environment variables.

cidr: The CIDR block for the VPC.

azs: Availability zones obtained from the data source.

public_subnets: List of public subnets.

private_subnets: List of private subnets.

map_public_ip_on_launch: Indicates whether instances launched in public subnets should have public IP addresses.

#### Database Subnets:

reate_database_subnet_group: Determines if a database subnet group should be created.

create_database_subnet_route_table: Determines if a route table for the database subnets should be created.

database_subnets: List of database subnets.

#### NAT Gateways - Outbound Communication:

enable_nat_gateway: Enables NAT gateways for outbound communication.

single_nat_gateway: Determines if a single NAT gateway should be used.

#### VPC DNS Parameters:

enable_dns_hostnames: Indicates whether instances launched in the VPC get DNS hostnames.

enable_dns_support: Indicates whether DNS resolution is supported for instances launched in the VPC.

#### VPC Tags:

tags and vpc_tags: Common tags applied to the VPC.

This module is designed to create a well-architected VPC with public and private subnets, NAT gateways, and other configurations based on best practices for AWS infrastructure. The specific details are customizable through the provided input variables.

## Create IAM role for EKS

![am](/pictures/iam.png)
![role](/pictures/role.png)

aws_iam_role Resource:
This resource creates an IAM role named ${local.name}-eks-master-role. The local.name variable is a combination of business_division and environment, providing a unique name for the role.

The assume_role_policy specifies who or what service can assume the role. In this case, it allows the Amazon EKS service (eks.amazonaws.com) to assume this role. This role is typically assumed by the EKS cluster's control plane.

aws_iam_role_policy_attachment Resources:
These resources attach IAM policies to the IAM role created earlier.

eks_AmazonEKSClusterPolicy attaches the AmazonEKSClusterPolicy IAM policy to the eks_master_role. This policy provides the necessary permissions for Amazon EKS cluster operations.

eks_AmazonEKSVPCResourceController attaches the AmazonEKSVPCResourceController IAM policy to the eks_master_role. This policy grants permissions needed for VPC resource controller operations.

![node](/pictures/node.png)
the terraform code above simply creates an IAM role for the node group and also attach policies to them

## create the kubernetes cluster on AWS

![k8s](/pictures/k8s.png)

This Terraform code above defines an AWS EKS (Amazon Elastic Kubernetes Service) cluster resource.

name: Specifies the name of the EKS cluster. It's constructed using the combination of business_division and environment (from local.name) along with cluster_name (a variable) to ensure uniqueness.

role_arn: Specifies the Amazon Resource Name (ARN) of the IAM role that grants permissions to the EKS cluster control plane. It references the IAM role created earlier for the EKS master node.

version: Specifies the Kubernetes version for the EKS cluster. This is determined by the cluster_version variable.

vpc_config: Specifies the VPC configuration for the EKS cluster.
subnet_ids: Specifies the IDs of the subnets where the EKS Elastic Network Interfaces (ENIs) will be created. It references the public subnets created by the VPC module.

endpoint_private_access: Specifies whether the cluster's Kubernetes API server can receive requests from within the VPC.

endpoint_public_access: Specifies whether the cluster's Kubernetes API server can receive requests from outside the VPC.

public_access_cidrs: Specifies the CIDR blocks that are allowed to access the Kubernetes API server if endpoint_public_access is enabled.

kubernetes_network_config: Specifies the Kubernetes network configuration for the cluster.

service_ipv4_cidr: Specifies the CIDR range for Kubernetes service IP addresses.

enabled_cluster_log_types: Specifies the types of logs to enable for the EKS cluster control plane. In this case, API server logs, audit logs, authenticator logs, controller manager logs, and scheduler logs are enabled.

depends_on: Specifies dependencies for the EKS cluster resource. It ensures that IAM role permissions are created before and deleted after handling the EKS cluster. This is important for proper management of EKS-managed EC2 infrastructure such as security groups.

## tfvars

A couple of .tfvars was also created.

A .tfvars file in Terraform is used to set and override input variables for your Terraform configurations. These files are often used to store sensitive or environment-specific values separately from your main Terraform configuration files.
