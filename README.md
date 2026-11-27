# AWS EKS Cluster with Terraform & Ansible

A DevOps  project that provisions a production-ready Amazon EKS cluster on AWS using **Terraform**, then automates application deployment with **Ansible**.
Project demonstrates how to automate the deployment of a complete application environment using Terraform and Ansible.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS (ap-southeast-2)                 │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   myapp-vpc                          │   │
│  │                                                      │   │
│  │  ┌─────────────────┐    ┌─────────────────────────┐  │   │
│  │  │  Public Subnets │    │    Private Subnets      │  │   │
│  │  │  (Load Balancer)│    │  (EKS Managed Nodes)    │  │   │
│  │  └────────┬────────┘    └───────────┬─────────────┘  │   │
│  │           │                         │                │   │
│  │      NAT Gateway              EKS Cluster            │   │
│  │                            (myapp-eks-cluster)       │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Tech Stack

| Tool        | Version   | Purpose                              |
|-------------|-----------|--------------------------------------|
| Terraform   | ~> 6.0 (AWS provider) | Infrastructure provisioning |
| AWS EKS     | 1.33      | Managed Kubernetes cluster           |
| Ansible     | latest    | Configuration management & deployment |
| Docker      | latest    | Containerisation                     |
| Kubernetes  | 1.33      | Container orchestration              |


## Infrastructure (Terraform)

### VPC (`vpc.tf`)
- CIDR block and subnets defined via variables
- NAT gateway enabled (single, for cost efficiency)
- DNS hostnames enabled
- Subnets tagged for EKS load balancer discovery

### EKS Cluster (`eks-cluster.tf`)
- Cluster name: `myapp-eks-cluster`
- Kubernetes version: `1.33`
- Public API endpoint access
- Managed node group: `t2.small` instances (1–3 nodes, desired: 3)
- Default add-ons: CoreDNS, kube-proxy, vpc-cni, eks-pod-identity-agent

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/install) >= 1.0
- [AWS CLI](https://aws.amazon.com/cli/) configured with appropriate credentials
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/) >= 2.12
- `kubectl` configured to connect to the EKS cluster

## Usage

### 1. Provision Infrastructure

https://github.com/MikoMaksM/aws-eks-terraform

```bash
cd terraform

# Create terraform.tfvars with your values:
# vpc_cidr_block = "10.0.0.0/16"
# private_subnet_cidr_blocks = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
# public_subnet_cidr_blocks  = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]

terraform init
terraform plan
terraform apply
```

### 2. Update kubeconfig

```bash
aws eks update-kubeconfig --region ap-southeast-2 --name myapp-eks-cluster
```

### 3. Deploy Application to EKS

```bash
cd ansible
ansible-playbook deploy-to-k8s.yaml
```

This creates the `my-app` namespace and deploys an nginx Deployment + LoadBalancer Service.

### 4. Deploy Docker on EC2 Instances

```bash
cd ansible
ansible-playbook -i aws_inventory_ec2.yaml deploy-docker-with-roles.yaml
```

### 5. Deploy Nexus Repository Manager

```bash
cd ansible
ansible-playbook deploy-nexus.yaml -i <inventory>
```

## Variables

| Variable                     | Description                        |
|------------------------------|------------------------------------|
| `vpc_cidr_block`             | CIDR block for the VPC             |
| `private_subnet_cidr_blocks` | List of private subnet CIDR blocks |
| `public_subnet_cidr_blocks`  | List of public subnet CIDR blocks  |

## Security Notes

- `terraform.tfvars` is excluded from version control — never commit secrets or credentials
- Terraform state files are gitignored — use a remote backend (e.g. S3) in production
- EKS API endpoint is publicly accessible; restrict `public_access_cidrs` in production environments

## Cleanup

```bash
cd terraform
terraform destroy
```

