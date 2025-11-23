# 🚀 Todo App – EKS GitOps Deployment

### **(ArgoCD + Terraform + CircleCI)**

A complete **production-ready GitOps pipeline** for deploying a **React Todo Application** on **Amazon EKS**, fully automated using **Terraform**, continuously delivered through **ArgoCD**, and integrated with **CircleCI** for CI/CD.
This repository showcases the *entire DevOps lifecycle*: containerization → infrastructure → Kubernetes workloads → GitOps automation → CI/CD pipeline.

---

## 🏗️ Architecture Overview

This project implements a modern **cloud-native** architecture:

* ⚛️ React Frontend
* 🐳 Docker Multi-Stage Build
* ☸️ Amazon EKS (Managed Kubernetes)
* 🟪 Terraform (Infrastructure as Code)
* 🎯 ArgoCD (GitOps Deployment Control Plane)
* 🔄 CircleCI (CI/CD Automation)
* 📦 DockerHub Image Registry
* 🧩 GitHub Repositories (App + Manifests)

---

## 🐳 Local Docker Testing

### **🔧 Multi-Stage Dockerfile**

* **Stage 1:** Build React app using Node
* **Stage 2:** Serve production build using Nginx

### **Commands**

```bash
docker build -t sanketdesai09/todo-react-app:latest .
docker run -d -p 3000:80 sanketdesai09/todo-react-app:latest
```

### **⚠️ Issues Found & Resolved**

| Issue                  | Cause                     | Fix                             |
| ---------------------- | ------------------------- | ------------------------------- |
| `package.json` missing | Missing dependencies      | Ran `npm install`               |
| Lockfile mismatch      | Corrupted dependency tree | Regenerated `package-lock.json` |

---

## 🌐 Infrastructure Deployment (Terraform)

### **🏗️ Components Provisioned**

* VPC (Public + Private Subnets)
* Internet Gateway & NAT Gateway
* EKS Cluster (v1.28)
* Managed Node Group
* IAM Roles & Security Groups
* Remote Backend (S3 + DynamoDB)

### **🛠️ Commands**

```bash
terraform init
terraform validate
terraform plan
terraform apply --auto-approve
```

### **🔍 Verify EKS Cluster**

```bash
aws eks --region us-east-1 update-kubeconfig --name todo-eks-cluster
kubectl get nodes
```

---

## ☸️ Kubernetes Deployment

### **📦 Apply Manifests**

```bash
kubectl apply -f k8s-manifests/deployment.yaml
kubectl apply -f k8s-manifests/service.yaml
kubectl apply -f k8s-manifests/hpa.yaml
```

### **🔍 Verify Resources**

```bash
kubectl get pods
kubectl get svc
```

---

## 🎯 ArgoCD Setup (GitOps Control Plane)

### **⚙️ Install ArgoCD**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### **🌍 Expose ArgoCD via LoadBalancer**

```bash
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

### ⚠️ **Issue:** ArgoCD created **internal** LoadBalancer

**Fix: Force public LB**

```bash
kubectl patch svc argocd-server -n argocd \
  -p '{
    "metadata": {
      "annotations": {
        "service.beta.kubernetes.io/aws-load-balancer-scheme": "internet-facing"
      }
    },
    "spec": { "type": "LoadBalancer" }
  }'
```

### **🔐 Retrieve ArgoCD Admin Password**

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

### **🧩 Configure ArgoCD Application**

* Connect manifests repository
* Enable:

  * Auto-Sync
  * Self-Healing
  * Prune Resources

---

## 🔄 CircleCI – CI/CD Automation

### **🛠️ Workflow Steps**

1. Build Docker image
2. Push image to DockerHub
3. Clone Kubernetes manifests repo
4. Update image tag via `sed`
5. Commit & push updated manifests
6. ArgoCD auto-sync deploys latest version

### **🔑 Required Environment Variables**

| Variable             | Purpose                       |
| -------------------- | ----------------------------- |
| `DOCKERHUB_USERNAME` | DockerHub login               |
| `DOCKERHUB_PASSWORD` | DockerHub token               |
| `GITHUB_TOKEN`       | Push changes to GitHub        |
| `MANIFEST_REPO`      | Kubernetes manifests repo URL |

---

## 📸 Screenshots

### **1. Docker Build & Local Testing**

* Build output
  <img width="1884" height="119" alt="Screenshot 2025-11-21 152824" src="https://github.com/user-attachments/assets/dddfe984-c3a4-446a-a686-c4d7ffcf09f0" />

* Running container
  <img width="1879" height="160" alt="Screenshot 2025-11-21 152800" src="https://github.com/user-attachments/assets/87109daf-25e8-4d16-ad64-f3d19dd947b4" />

* Application in browser
  <img width="1920" height="927" alt="todo-gitops-local" src="https://github.com/user-attachments/assets/57d2cd9c-4070-47a6-b4b8-2bbdd57adb28" />


### **2. Terraform Infrastructure**

* Terraform plan
  <img width="807" height="250" alt="Screenshot 2025-11-21 160223" src="https://github.com/user-attachments/assets/1d0ff54d-6c86-4bad-8fef-25a3cc606370" />

* Terraform apply
  <img width="1174" height="238" alt="Screenshot 2025-11-21 161504" src="https://github.com/user-attachments/assets/5c142fca-7bfe-4835-8ed9-fad87372daec" />
  

### **3. Kubernetes Deployment**

* Pods running
  <img width="1002" height="93" alt="Screenshot 2025-11-23 103333" src="https://github.com/user-attachments/assets/bfbc155f-c32c-4bdd-b445-efb768f52b41" />

* Service details
  <img width="1761" height="95" alt="image" src="https://github.com/user-attachments/assets/ae51ab84-1841-4767-951e-bebfa334aef2" />


### **4. ArgoCD Setup**

* ArgoCD pods
  <img width="1112" height="211" alt="Screenshot 2025-11-23 103528" src="https://github.com/user-attachments/assets/cc47ef10-8597-4fad-910e-bb4a5e195104" />

* LoadBalancer details
  <img width="1648" height="242" alt="Screenshot 2025-11-21 174218" src="https://github.com/user-attachments/assets/813d9658-0fc2-495d-ad4f-997b0b532343" />

* App syncing
  <img width="1437" height="461" alt="Screenshot 2025-11-21 173548" src="https://github.com/user-attachments/assets/a3ff1f26-05c1-416b-9d44-f3f8cfd7cd60" />


### **5. CircleCI Pipeline**

* Successful workflow
  <img width="1659" height="837" alt="Screenshot 2025-11-21 173649" src="https://github.com/user-attachments/assets/173dcf20-7e0c-49c2-af56-abaf030e94ac" />

* Image pushed to registry
  <img width="951" height="562" alt="Screenshot 2025-11-21 173746" src="https://github.com/user-attachments/assets/bfcd5197-7a16-48ce-834f-f07cf7280019" />


### **6. Running Application**

* Before adding tasks
  <img width="1820" height="347" alt="Screenshot 2025-11-21 173504" src="https://github.com/user-attachments/assets/f96f4ef0-687f-4a56-a586-86b8cfe10d50" />

* After adding tasks
  <img width="1815" height="417" alt="Screenshot 2025-11-21 173527" src="https://github.com/user-attachments/assets/effbc0f7-e527-4b03-981b-c95029ac8d5a" />

---

## 👤 Author

**Riddhesh Sutar**

*DevOps & AWS Engineer*

GitOps | Kubernetes | Terraform | AWS | CI/CD | Cloud Infrastructure

---
