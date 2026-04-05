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

To ensure data persistence for our PostgreSQL database across pod restarts and failures, we configured an **AWS Elastic Block Store (EBS)** volume via Kubernetes Persistent Volume Claims (PVC). 

We utilized the **EBS CSI (Container Storage Interface) Driver** to allow Kubernetes to provision, manage, and attach EBS volumes dynamically for our stateful workloads on the EKS cluster.

---

Built with 💜 by the Jerney team. No cap, this blog platform hits different. 🛤️
