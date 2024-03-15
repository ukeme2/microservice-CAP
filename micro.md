# Deploying the sock-shop application to the k8s cluster & setting up prometheus

## Ingress-rule

### micro.tf;

![mi](/pictures/micro.png)

This code defines a Kubernetes Ingress resource named "micro-ingress" for the "sock-shop" application within the "sock-shop" namespace. The Ingress resource is used to configure external access to services within a Kubernetes cluster.

Metadata:

Name: "sock-shop"

Namespace: "sock-shop"

Labels: It assigns a label "name=front-end" to the Ingress resource.

Annotations: It sets the annotation "kubernetes.io/ingress.class" to "nginx", indicating that this Ingress should be handled by an Ingress controller with the class "nginx".

Specification (Spec):

Rule: Defines a routing rule for incoming requests.

Host: Specifies the host for which the Ingress should route traffic. In this case, it's "sock-shop.ukeme.live".

HTTP: Defines HTTP-based routing.

Path: Defines the path-based routing.

Backend: Specifies the backend service to which traffic should be routed.

Service:
Name: "front-end" specifies the name of the Kubernetes service that should receive the traffic.

Port: Specifies the port number (80 in this case) on which the service is listening.

Overall, this Ingress resource configures routing for the "sock-shop" application, directing incoming HTTP traffic for the host "sock-shop.ukeme.live" to the service named "front-end" within the same namespace. The annotation suggests that an Ingress controller compatible with the "nginx" class should handle this routing.

## prome.tf

![pro](/pictures/prome.png)

This code defines another Kubernetes Ingress resource, named "prome-ingress", for handling incoming traffic to a service within the "prometheus" namespace.

## provider.tf

![ro](/pictures/pro.png)

This Terraform configuration script appears to set up providers for managing resources in an AWS Elastic Kubernetes Service (EKS) cluster and deploying Helm charts onto it.

Terraform Block:

Specifies required providers (helm, kubernetes, kubectl) along with their versions and sources.
Data Blocks:

aws_eks_cluster: Retrieves information about an EKS cluster named "hr-dev-eks-demo".

aws_eks_cluster_auth: Retrieves authentication information for the same EKS cluster.
Provider Blocks:

aws: Configures the AWS provider with the specified region (us-east-1).

helm: Configures the Helm provider to interact with Kubernetes clusters. It specifies the Kubernetes configuration (~/.kube/config) to connect to the cluster. However, the host, cluster_ca_certificate, and token are commented out, likely because the values are provided by aws_eks_cluster and aws_eks_cluster_auth data sources.

kubernetes: Configures the Kubernetes provider to interact with Kubernetes clusters. Similar to the Helm provider, it specifies the Kubernetes configuration path but leaves out the host, cluster_ca_certificate, and token.

kubectl: Configures the kubectl provider, which allows running kubectl commands via Terraform. It specifies the endpoint, certificate authority, and token to authenticate with the EKS cluster. The load_config_file is set to false, indicating that Terraform will use the provided configuration instead of loading from a file. It's also aliased as "aws."

## Microservice

The microservice folder consist of code for each services of the sock-shop application which includes; front-end, carts, payments etc and can be gotten from the micro-service demo on github.

## deploying nginx-controller using helm

![ngin](/pictures/resource.png)
![nx](/pictures/sett.png)
![bm](/pictures/helm.png)

This Terraform configuration sets up infrastructure components on an AWS EKS cluster, including creating a namespace, deploying an NGINX Ingress Controller using Helm, and configuring various settings for the Ingress Controller.

Here's a breakdown of what each part of the configuration does:

time_sleep Resource:

This resource creates a wait time of 20 seconds to allow the AWS EKS cluster to be fully provisioned before proceeding with further resource creation.

It depends on the AWS EKS cluster data source to ensure it waits for the cluster to be ready.
kubernetes_namespace Resource:

This resource creates a Kubernetes namespace named "nginx-ingress".
It depends on the "time_sleep.wait_for_kubernetes" resource to ensure it waits for the cluster to be ready before creating the namespace.

helm_release Resource (Ingress Controller):

This resource deploys the NGINX Ingress Controller using Helm onto the Kubernetes cluster.

It depends on both the "kubernetes_namespace.nginx-namespace" resource and the "time_sleep.wait_for_kubernetes" resource.
The NGINX Ingress Controller is deployed from the specified repository with the chart version "4.5.2".
It specifies the namespace where the Ingress Controller should be deployed.
Various configuration values are set for the Ingress Controller using the values and set blocks. These configurations include:
Overriding the fullname to "load".

Setting the controller name to "nginx".

Configuring the service type as "LoadBalancer".

Enabling pod security policy.

Disabling persistent volume.

Setting resource limits for CPU and memory.

Providing a map of values for server resources using yamlencode.

## configuring DNS settings using Route 53:

![d](/pictures/null.png)
![n](/pictures/lacal%20copy.png)
![s](/pictures/cert.png)
![d](/pictures/acm.png)

null_resource "get_nlb_hostname":

This null_resource uses a local-exec provisioner to execute commands locally.

It runs AWS CLI commands to update the Kubernetes configuration and retrieve the hostname of the load balancer associated with the NGINX Ingress Controller service.

The retrieved hostname is stored in a text file named "lb_hostname.txt" within the module directory.

data "local_file" "lb_hostname":

This data source reads the content of the "lb_hostname.txt" file generated by the null_resource.

It ensures that Terraform waits for the null_resource to complete before reading the hostname from the file.

aws_route53_zone "hosted_zone":

This resource creates a Route 53 hosted zone with the specified domain name.
Tags are added to the hosted zone for identification purposes.

locals:

This block defines a map named "instances" containing the DNS names for different services.

These DNS names are constructed based on the provided domain name variable.

aws_route53_record "C-record":

This resource creates CNAME records in the Route 53 hosted zone.

It iterates over the instances defined in the locals block and associates each DNS name with the load balancer hostname retrieved earlier.

The CNAME records point to the load balancer hostname.

aws_acm_certificate:

This resource requests a public certificate from AWS ACM (Amazon Certificate Manager).

It specifies the primary domain name and any additional alternative domain names for the certificate.

The validation method is set to "DNS", meaning that DNS records will be used for certificate validation.

aws_route53_record "route53_record":

This resource creates DNS records in Route 53 for certificate validation.

It creates records based on the domain validation options provided by AWS ACM.

These records are necessary for AWS to verify domain ownership during certificate issuance.

aws_acm_certificate_validation:

This resource validates the ACM certificate by providing validation records to AWS ACM.

It uses the certificate ARN and the fully qualified domain names (FQDNs) of the validation records created earlier.

In summary, this Terraform configuration automates the setup of DNS records in Route 53, associates them with a load balancer hostname, and manages the validation of an ACM certificate for the specified domain names.

## setting up providers and data sources for interacting with AWS EKS

![t](/pictures/t-helm.png)
![t2](/pictures/t-helm2.png)

Terraform Block with Required Providers:

It declares the required Terraform providers and their versions. These providers are:

Helm: Used for managing Helm charts.

Kubernetes: Used for interacting with Kubernetes resources.

Kubectl: Used for managing Kubernetes clusters using kubectl commands.

Data Blocks for AWS EKS Cluster and Authentication:

data "aws_eks_cluster" "hr-dev-eks-demo": Fetches data about the AWS EKS cluster named "hr-dev-eks-demo".
data "aws_eks_cluster_auth" "hr-dev-eks-demo_auth": Fetches authentication data for the AWS EKS cluster named "hr-dev-eks-demo".

Provider Block for AWS:

Specifies the AWS provider configuration with the region set to "eu-west-2".

Provider Block for Helm:

Configures the Helm provider to interact with the Kubernetes cluster.
Sets the config_path to "~/.kube/config", which points to the Kubernetes config file. However, the host, cluster CA certificate, and token are commented out.

Provider Block for Kubernetes:

Configures the Kubernetes provider to interact with the Kubernetes cluster.
Sets the config_path to "~/.kube/config", and other authentication details are commented out.

Provider Block for Kubectl:

Configures the Kubectl provider to manage the Kubernetes cluster.
Sets the config_path to "~/.kube/config" and specifies the cluster's endpoint, cluster CA certificate, and authentication token using data fetched from AWS EKS.

This configuration essentially sets up the necessary providers and authentication details for interacting with an AWS EKS cluster using Terraform, Helm, and kubectl commands.
