# 🚀 EKS Setup & Deployment using Terraform

This guide covers:

* EKS cluster creation using Terraform
* Connecting using AWS CLI
* Installing kubectl (Windows)
* Deploying NGINX on Kubernetes
* Exposing via LoadBalancer
* Cleanup (end-to-end)

---

# 📦 Prerequisites

* AWS Account
* AWS CLI installed
* Terraform (`>= 1.6`)
* kubectl (we will install below)
* IAM user with required permissions

---

# 🔐 Configure AWS CLI

```bash
aws configure
```

---

# ⚙️ Install kubectl (Windows)

## 🚀 Step 1: Download

```powershell
curl.exe -LO "https://dl.k8s.io/release/v1.33.0/bin/windows/amd64/kubectl.exe"
```

## 📁 Step 2: Move

```powershell
mkdir C:\kubectl
move kubectl.exe C:\kubectl\
```

## 🔧 Step 3: Add to PATH

Add to System PATH:

```
C:\kubectl
```

Restart terminal.

## ✅ Step 4: Verify

```bash
kubectl version --client
```

---

# 🪣 Backend Setup (S3 + DynamoDB)

```bash
aws s3api create-bucket --bucket my-terraform-state-eks-demo --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1

aws s3api put-bucket-versioning --bucket my-terraform-state-eks-demo --versioning-configuration Status=Enabled

aws dynamodb create-table --table-name terraform-lock --attribute-definitions AttributeName=LockID,AttributeType=S --key-schema AttributeName=LockID,KeyType=HASH --billing-mode PAY_PER_REQUEST --region ap-south-1
```

---

# 🏗️ Deploy EKS

```bash
cd terraform/k8
terraform init
terraform apply -auto-approve
```

---

# 🔗 Connect to Cluster

```bash
aws eks update-kubeconfig --region ap-south-1 --name demo-eks-webapp-v25
```

---

# ✅ Verify Cluster

```bash
kubectl get nodes
kubectl get pods -A
```

---

# 🚀 Deploy NGINX

```bash
kubectl create deployment nginx --image=nginx
kubectl get pods
```

---

# 🌐 Expose Publicly

```bash
kubectl expose deployment nginx --type=LoadBalancer --port=80
kubectl get svc nginx
```

Access:

```
http://<EXTERNAL-IP>
```

---

# ⚡ Debug Commands

```bash
kubectl logs <pod>
kubectl describe pod <pod>
kubectl get events
kubectl exec -it <pod> -- /bin/bash
```

---

# 🧹 Cleanup

```bash
kubectl delete svc nginx
kubectl delete deployment nginx
```

---

# 💣 Destroy Infra

```bash
terraform destroy -auto-approve
```

---

# 🎯 Summary

| Step    | Command                   |
| ------- | ------------------------- |
| Apply   | terraform apply           |
| Connect | aws eks update-kubeconfig |
| Deploy  | kubectl create deployment |
| Expose  | kubectl expose            |
| Delete  | kubectl delete            |
| Destroy | terraform destroy         |

---
