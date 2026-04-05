# 🛤️ Jerney — Blog Platform

A Gen-Z vibe blog platform built with a 3-tier architecture — React frontend, Node.js backend, and PostgreSQL database.

![Tech Stack](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react)
![Tech Stack](https://img.shields.io/badge/Node.js-20-339933?style=flat-square&logo=node.js)
![Tech Stack](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql)
![Tech Stack](https://img.shields.io/badge/Docker-24.0-2496ED?style=flat-square&logo=docker)
![Tech Stack](https://img.shields.io/badge/Kubernetes-1.28-326CE5?style=flat-square&logo=kubernetes)

---

## 🌟 Project Overview

Jerney is a fully-functional blog platform that embraces modern design and seamless user experience. It's built to allow users to create, read, update, and delete (CRUD) blog posts in an interactive, responsive, and visually appealing environment.

### ✨ Features
- 📝 Create blog posts with emoji vibes
- ✏️ Edit your existing posts
- 🗑️ Delete posts you're not feeling anymore
- 💬 Comment on posts
- 🎨 Gen-Z dark UI with glassmorphism and gradients

## 🏗️ Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│   Backend    │────▶│  PostgreSQL   │
│   (React +   │◀────│  (Node.js +  │◀────│              │
│    Nginx)    │     │   Express)   │     │              │
│   Port 80    │     │  Port 5000   │     │  Port 5432   │
└──────────────┘     └──────────────┘     └──────────────┘
```

## 📁 Project Structure

```
Jerney/
├── frontend/                # React (Vite) frontend with its own Dockerfile
├── backend/                 # Node.js Express API with its own Dockerfile
├── kubernetes/              # Kubernetes manifests
├── docker-compose.yml       # Local development setup
└── README.md
```

## 🚀 DevOps Implementation

This project implements robust DevOps practices for containerization, orchestration, and seamless deployments.

### 🐳 Docker & Docker Compose
Both the frontend and backend are fully containerized using **Dockerfiles**. We use **Docker Compose** for local deployment and testing, spinning up the frontend, backend, and PostgreSQL database with a single unified command.

```bash
# Run locally with Docker Compose
docker-compose up -d
```

### ☸️ Kubernetes (Minikube Deployment)
The application was initially deployed and tested on a local Kubernetes cluster using **Minikube**.

**Minikube Deployment Steps:**

1. Start your Minikube cluster:
   ```bash
   minikube start
   ```
2. Apply the Kubernetes manifests from the `kubernetes/` directory:
   ```bash
   # Apply namespace and secrets
   kubectl apply -f kubernetes/namespace.yml
   kubectl apply -f kubernetes/secrets.yml
   
   # Apply storage and database
   kubectl apply -f kubernetes/pv.yml
   kubectl apply -f kubernetes/pvc.yml
   kubectl apply -f kubernetes/db-deployment.yml
   kubectl apply -f kubernetes/db-service.yml
   
   # Apply backend and frontend
   kubectl apply -f kubernetes/backend-deployment.yml
   kubectl apply -f kubernetes/backend-service.yml
   kubectl apply -f kubernetes/frontend-deployment.yml
   kubectl apply -f kubernetes/frontend-service.yml
   ```
3. Check the status of your pods and services:
   ```bash
   kubectl get pods -n <your-namespace>
   kubectl get svc -n <your-namespace>
   ```
4. Access the application using Minikube tunnel or node port:
   ```bash
   minikube service frontend-service -n <your-namespace>
   ```

### ☁️ AWS EKS Deployment with EBS
For a production-grade setup, the application is deployed on Amazon Elastic Kubernetes Service (EKS). 

**EKS Deployment Steps:**

1. **Install Prerequisites & Configure AWS**
   Ensure you have `aws-cli`, `eksctl`, and `kubectl` installed.
   ```bash
   aws configure
   ```

2. **Create EKS Cluster**
   Create a cluster with 2 `t3.medium` nodes in the `ap-south-1` region.
   *(Note: `t3.medium` instances are recommended here; smaller instances like `t3.small` may lack the necessary resources to run all application workloads and cluster add-ons smoothly.)*
   ```bash
   eksctl create cluster --name jerney-cluster --region ap-south-1 --nodegroup-name standard-workers --node-type t3.medium --nodes 2
   ```

3. **Configure EBS CSI Driver (For Persistent Storage)**
   To ensure data persistence for our PostgreSQL database across pod restarts, we configure an AWS Elastic Block Store (EBS) volume.
   ```bash
   # Create an IAM OIDC provider for the cluster
   eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster jerney-cluster --approve
   
   # Create IAM role and attach the EBS CSI policy
   eksctl create iamserviceaccount \
     --name ebs-csi-controller-sa \
     --namespace kube-system \
     --cluster jerney-cluster \
     --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
     --approve \
     --role-only \
     --role-name AmazonEKS_EBS_CSI_DriverRole
     
   # Install the EBS CSI Driver as an EKS Add-on
   eksctl create addon --name aws-ebs-csi-driver --cluster jerney-cluster --service-account-role-arn arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/AmazonEKS_EBS_CSI_DriverRole --force
   ```

4. **Deploy the Application**
   On EKS, use a `StorageClass` to dynamically provision EBS volumes instead of manually creating a `PersistentVolume` (`pv.yml`).
   ```bash
   kubectl apply -f kubernetes/namespace.yml
   kubectl apply -f kubernetes/secrets.yml
   
   # Note: apply storageclass.yml instead of pv.yml for dynamic provisioning
   kubectl apply -f kubernetes/storageclass.yml
   kubectl apply -f kubernetes/pvc.yml
   kubectl apply -f kubernetes/db-deployment.yml
   kubectl apply -f kubernetes/db-service.yml
   
   kubectl apply -f kubernetes/backend-deployment.yml
   kubectl apply -f kubernetes/backend-service.yml
   kubectl apply -f kubernetes/frontend-deployment.yml
   kubectl apply -f kubernetes/frontend-service.yml
   ```

5. **Install AWS Load Balancer Controller**
   To securely expose our application to the internet, we configure an Application Load Balancer (ALB) via an Ingress resource.
   ```bash
   # Download the IAM policy for the AWS Load Balancer Controller
   curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
   aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
   
   # Create a service account mapped to the IAM role
   eksctl create iamserviceaccount \
     --cluster=jerney-cluster \
     --namespace=kube-system \
     --name=aws-load-balancer-controller \
     --role-name AmazonEKSLoadBalancerControllerRole \
     --attach-policy-arn=arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):policy/AWSLoadBalancerControllerIAMPolicy \
     --approve
     
   # Install the controller using Helm
   helm repo add eks https://aws.github.io/eks-charts
   helm repo update eks
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
     -n kube-system \
     --set clusterName=jerney-cluster \
     --set serviceAccount.create=false \
     --set serviceAccount.name=aws-load-balancer-controller
   ```

6. **Apply Ingress**
   Finally, apply the Ingress rule to expose the frontend service externally.
   ```bash
   kubectl apply -f kubernetes/ingress.yml
   ```

---

Built with 💜 by the Jerney team. No cap, this blog platform hits different. 🛤️
