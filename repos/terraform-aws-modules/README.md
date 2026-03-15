# terraform-aws-modules

Production-grade, reusable Terraform modules for AWS — built from real-world
multi-environment provisioning across 4 environments (dev / staging / pre-prod / prod).

## What's inside

```
terraform-aws-modules/
├── modules/
│   ├── vpc/              # VPC, subnets, route tables, NACLs, IGW, NAT
│   ├── eks/              # EKS cluster, node groups, RBAC, IRSA
│   ├── rds/              # RDS / Aurora with subnet groups, parameter groups
│   ├── iam/              # Roles, policies, least-privilege patterns
│   ├── alb/              # Application Load Balancer, target groups, listeners
│   ├── ecr/              # ECR repository with lifecycle policies
│   └── secrets-manager/  # Secrets with IAM bindings and rotation config
├── environments/
│   ├── dev/
│   ├── staging/
│   ├── pre-prod/
│   └── prod/
├── backend.tf            # S3 remote state + DynamoDB locking
└── README.md
```

## Key design principles

- **Remote state** — S3 backend with DynamoDB state locking per environment
- **Workspace isolation** — separate `.tfvars` per environment, no shared state
- **Least-privilege IAM** — every module ships with a minimal IAM policy
- **No hardcoded values** — all configurable via input variables with validation
- **OPA policy gates** — `terraform plan` blocked if policies fail

## Usage

```hcl
module "vpc" {
  source = "../../modules/vpc"

  name             = "prod-vpc"
  cidr             = "10.0.0.0/16"
  azs              = ["ap-south-1a", "ap-south-1b", "ap-south-1c"]
  private_subnets  = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets   = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  enable_nat_gateway = true
}

module "eks" {
  source = "../../modules/eks"

  cluster_name    = "prod-cluster"
  cluster_version = "1.30"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnet_ids
  node_instance_types = ["m5.xlarge"]
  desired_size    = 3
  min_size        = 2
  max_size        = 10
}
```

## Remote state setup

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "thaneesh-tf-state"
    key            = "prod/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

## Results

- Reduced new-environment provisioning from **~6 hours to under 30 minutes**
- Eliminated manual configuration drift across 4 environments
- Adopted as team-wide standard — cut new module development time by **~50%**
