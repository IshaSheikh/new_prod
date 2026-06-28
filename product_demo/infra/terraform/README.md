# Infrastructure (Terraform)

This directory contains the Terraform configuration for provisioning the AWS infrastructure for the School SaaS Platform.

## Overview

We use AWS as our primary cloud provider. The infrastructure is provisioned using Infrastructure as Code (IaC) principles.

- **Compute**: ECS Fargate for API and Web containers.
- **Database**: RDS PostgreSQL 16 with Multi-AZ.
- **Storage**: S3 for file uploads with CloudFront CDN.
- **Networking**: VPC, public/private subnets, NAT Gateway, ALB.

## Prerequisites

- Terraform >= 1.5.0
- AWS CLI configured with administrator access
- AWS account

## Environments

State files are stored in an S3 backend (`schoolsaas-terraform-state`). Environments are separated using Terraform workspaces or separate state files per environment (`staging`, `prod`).

## Deployment

Changes to Terraform should be reviewed via PR. Apply operations are typically run locally by DevOps or via secure CI/CD runners with appropriate IAM roles.

```bash
terraform init
terraform plan -var-file="env/staging.tfvars"
terraform apply -var-file="env/staging.tfvars"
```

For more details on the architecture and deployment pipeline, see [CI/CD Documentation](../../docs/06-engineering/ci-cd.md).
