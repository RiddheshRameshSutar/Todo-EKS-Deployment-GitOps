# 🚀 AWS EKS Infrastructure using Terraform

## Overview
This Terraform configuration automates the creation of a complete AWS EKS environment for deploying containerized applications like a Node.js ToDo App.

The setup includes:
- **Custom VPC** with public/private subnets and NAT gateway  
- **EKS Cluster** (control plane)  
- **EC2 Worker Nodes** (`t2.medium`) for workloads  
- **Outputs** to easily connect kubectl to the cluster

---

## 📁 Folder Structure
eks-infra/
├── backend.tf
├── main.tf
├── provider.tf
├── vpc.tf
├── eks-cluster.tf
├── nodegroup.tf
├── variables.tf
├── outputs.tf
└── README.md


---

## ⚙️ Prerequisites
- Terraform ≥ 1.5
- AWS CLI configured with IAM credentials
- kubectl installed
- eksctl (optional for validation)

---

## Create S3 bucket and DynamoDB lock table
- aws s3api create-bucket \
  --bucket todoapp-eks-terraform-state \
  --region us-east-1 \

- aws s3api put-bucket-versioning \
  --bucket todoapp-eks-terraform-state \
  --versioning-configuration Status=Enabled

- aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5

## 🧩 Deployment Steps

### 1️⃣ Initialize Terraform

terraform init

### 2️⃣ Validate and Plan
terraform validate
terraform plan

### 3️⃣ Apply Configuration
terraform apply -auto-approve

### 4️⃣ Update kubeconfig
aws eks --region us-east-1 update-kubeconfig --name todo-eks-cluster

### 5️⃣ Verify Cluster
kubectl get nodes
- You should see 2 worker nodes (t2.medium) in Ready state.

### 🧰 Key Configurations
| Component | Description |
| ......... | ........... |
| Region | us-east-1 |
| Cluster Name | todo-eks-cluster |
| Instance Type | t2.medium |
| Node Count | 2 desired (auto-scalable 1–4) |
|Subnets |	2 public, 2 private |
|NAT Gateway | Enabled (single) |

### 🧹 Destroy Infrastructure
terraform destroy -auto-approve

### 🧾 Notes
- Autoscaling for pods can be managed using Kubernetes HPA and Cluster Autoscaler.
- Integrate with ArgoCD for GitOps-based deployments.
- Recommended to store Terraform state in a remote backend (e.g., S3 with DynamoDB locking).

