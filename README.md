# Socks Shop Microservices Deployment

## Project overview:

The goal of this project is to deploy a microservices-based application using Infrastructure as Code (IaaC) on Kubernetes. This README provides a step-by-step guide to automate the deployment process, ensuring clarity, maintenance, and security.

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
