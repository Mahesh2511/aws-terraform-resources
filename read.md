# 🚀 EKS Setup & Deployment using Terraform

This guide covers:

* EKS cluster creation using Terraform
* Connecting using AWS CLI
* Deploying NGINX on Kubernetes
* Exposing via LoadBalancer
* Cleanup (destroy everything)

---

# 📦 Prerequisites

* AWS Account
* AWS CLI installed
* Terraform installed (`>= 1.6`)
* kubectl installed
* IAM user with required permissions

---

# 🔐 Configure AWS CLI

```bash
aws configure
```

Enter:

* Access Key
* Secret Key
* Region → `ap-south-1`

---

# 🏗️ Terraform Setup

## 📁 Directory Structure

```
terraform/
  └── k8/
       ├── main.tf
       ├── provider.tf
       ├── backend.tf
```

---

## 🪣 Create Backend (S3 + DynamoDB)

### Create S3 bucket

```bash
aws s3api create-bucket \
  --bucket my-terraform-state-eks-demo \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1
```

### Enable versioning

```bash
aws s3api put-bucket-versioning \
  --bucket my-terraform-state-eks-demo \
  --versioning-configuration Status=Enabled
```

### Create DynamoDB table

```bash
aws dynamodb create-table \
  --table-name terraform-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
```

---

## 🧾 backend.tf

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-eks-demo"
    key            = "eks/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```

---

# ☸️ EKS Terraform Configuration (main.tf)

```hcl
provider "aws" {
  region = "ap-south-1"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "eks-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    "kubernetes.io/cluster/demo-eks-webapp-v25" = "shared"
  }
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.0.0"

  cluster_name    = "demo-eks-webapp-v25"
  cluster_version = "1.33"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access = true
  enable_cluster_creator_admin_permissions = true

  cluster_encryption_config = {
    resources = ["secrets"]
  }

  eks_managed_node_groups = {
    default = {
      desired_size = 2
      min_size     = 1
      max_size     = 3

      instance_types = ["t3.small", "t3.medium"]
      ami_type       = "AL2023_x86_64_STANDARD"
    }
  }

  tags = {
    Environment = "dev"
  }
}
```

---

# 🚀 Deploy Infrastructure

```bash
cd terraform/k8

terraform init
terraform plan
terraform apply -auto-approve
```

⏱️ Takes ~15–20 minutes

---

# 🔗 Connect to EKS Cluster

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name demo-eks-webapp-v25
```

---

# ✅ Verify Cluster

```bash
kubectl get nodes
```

---

# 🚀 Deploy Sample App (NGINX)

## Create deployment

```bash
kubectl create deployment nginx --image=nginx
```

## Verify pods

```bash
kubectl get pods
```

---

# 🌐 Expose Application

```bash
kubectl expose deployment nginx --type=LoadBalancer --port=80
```

---

# 🔍 Get Public URL

```bash
kubectl get svc nginx
```

Wait until:

```
EXTERNAL-IP → available
```

---

# 🌍 Access from Browser

```
http://<EXTERNAL-IP>
```

---

# ⚠️ Cost Note

* LoadBalancer → charged hourly
* NAT Gateway → costly (~$30/month)

---

# 🧹 Cleanup Kubernetes Resources

```bash
kubectl delete svc nginx
kubectl delete deployment nginx
```

---

# 💣 Destroy Full Infrastructure

```bash
terraform destroy -auto-approve
```

---

# 🔍 Verify Cleanup

```bash
aws eks list-clusters --region ap-south-1
```

---

# ⚠️ Troubleshooting

## kubectl not found

Install:

```bash
choco install kubernetes-cli -y
# Alternative (manual install)
curl -LO "https://dl.k8s.io/release/v1.30.0/bin/windows/amd64/kubectl.exe"
move kubectl.exe C:\Windows\System32\
# OR add kubectl to PATH manually if not using System32
setx PATH "%PATH%;C:\path\to\kubectl"
```

---

## No external IP

* Wait 2–3 minutes
* Check:

```bash
kubectl get svc
```

---

## Node not ready

```bash
kubectl get nodes
```

---

# 🎯 Summary

| Step         | Command                     |
| ------------ | --------------------------- |
| Create infra | `terraform apply`           |
| Connect      | `aws eks update-kubeconfig` |
| Deploy app   | `kubectl create deployment` |
| Expose       | `kubectl expose`            |
| Access       | Browser via EXTERNAL-IP     |
| Cleanup      | `kubectl delete`            |
| Destroy      | `terraform destroy`         |

---

# 🚀 Next Steps

* Add Ingress (ALB)
* Setup domain + HTTPS
* CI/CD pipeline (build → deploy)

---
