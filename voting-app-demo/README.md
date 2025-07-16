# Kubernetes Voting App Demo

A simple distributed voting application built with microservices architecture and deployed on Kubernetes. This demo showcases how multiple services work together in a containerized environment.

## 🏗️ Application Architecture

This voting application consists of 5 microservices:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Voting App  │    │   Worker    │    │ Result App  │
│   (Web UI)  │    │ (Processor) │    │   (Web UI)  │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐           ┌─────────────────────────┐
│    Redis    │           │      PostgreSQL        │
│  (Queue)    │           │     (Database)         │
└─────────────┘           └─────────────────────────┘
```

### Components:
- **Voting App**: Frontend web application where users cast votes (Python Flask)
- **Redis**: In-memory database that stores votes temporarily
- **Worker**: Background service that processes votes from Redis to PostgreSQL (.NET)
- **PostgreSQL**: Persistent database that stores processed votes
- **Result App**: Frontend web application that displays voting results (Node.js)

## 📋 Prerequisites

### System Requirements:
- **Operating System**: Linux, macOS, or Windows
- **RAM**: Minimum 4GB (8GB recommended)
- **CPU**: 2 cores minimum
- **Disk Space**: At least 10GB free space

### Required Software:

#### 1. Docker
**What it is**: Platform for running containerized applications
**Installation**:
- **Linux**: `curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh`
- **macOS/Windows**: Download Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop)

#### 2. Kubernetes Cluster
Choose one of these options:

**Option A: Minikube (Recommended for beginners)**
```bash
# Install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start minikube
minikube start
```

**Option B: Docker Desktop Kubernetes**
- Enable Kubernetes in Docker Desktop settings

**Option C: Cloud Kubernetes (AWS EKS, Google GKE, Azure AKS)**
- Follow your cloud provider's setup guide

#### 3. kubectl
**What it is**: Command-line tool for interacting with Kubernetes
```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## 🚀 Quick Start Guide

### Step 1: Verify Prerequisites
```bash
# Check Docker
docker --version

# Check Kubernetes cluster
kubectl cluster-info

# Check if cluster is ready
kubectl get nodes
```

### Step 2: Clone and Navigate
```bash
# If you haven't already, navigate to the project directory
cd voting-app-demo
```

### Step 3: Deploy the Application
Deploy all components in the correct order:

```bash
# 1. Deploy Redis (Vote storage)
kubectl apply -f redis-pod.yaml
kubectl apply -f redis-service.yaml

# 2. Deploy PostgreSQL (Results database)
kubectl apply -f postgres-pod.yaml
kubectl apply -f postgres-service.yaml

# 3. Deploy Voting App (Frontend)
kubectl apply -f voting-app-pod.yaml
kubectl apply -f voting-app-serice.yaml

# 4. Deploy Result App (Results frontend)
kubectl apply -f result-app-pod.yaml
kubectl apply -f result-app-service.yaml

# 5. Deploy Worker (Vote processor)
kubectl apply -f worker-pod.yaml
```

### Step 4: Verify Deployment
```bash
# Check if all pods are running
kubectl get pods

# Check services
kubectl get services

# Wait for all pods to be in 'Running' status
kubectl get pods -w
```

### Step 5: Access the Application

#### Get Access URLs:
```bash
# For Minikube users
minikube service voting-service --url
minikube service result-service --url

# For other setups, get node IP
kubectl get nodes -o wide
```

#### Access Points:
- **Voting Interface**: `http://<NODE-IP>:30004`
- **Results Interface**: `http://<NODE-IP>:30005`

## 🚀 Deploying using Deployment YAMLs and Service YAMLs

### Alternative Deployment Method (Recommended for Production)

Instead of using individual pods, you can deploy using Kubernetes Deployments which provide better scalability, rolling updates, and self-healing capabilities.

#### Step 1: Deploy using Deployment Files
```bash
# 1. Deploy Redis using Deployment
kubectl apply -f deployment-yaml/redis-deploy.yaml
kubectl apply -f redis-service.yaml

# 2. Deploy PostgreSQL using Deployment
kubectl apply -f deployment-yaml/postgres-deploy.yaml
kubectl apply -f postgres-service.yaml

# 3. Deploy Voting App using Deployment
kubectl apply -f deployment-yaml/voting-app-depoloy.yaml
kubectl apply -f voting-app-serice.yaml

# 4. Deploy Result App using Deployment
kubectl apply -f deployment-yaml/result-app-deploy.yaml
kubectl apply -f result-app-service.yaml

# 5. Deploy Worker using Deployment
kubectl apply -f deployment-yaml/worker-app-deploy.yaml
```

#### Step 2: Verify Deployment Status
```bash
# Check deployments
kubectl get deployments

# Check pods created by deployments
kubectl get pods

# Check services
kubectl get services
```

#### Benefits of Using Deployments:
- **Scalability**: Easily scale replicas up or down
- **Rolling Updates**: Update applications without downtime
- **Self-Healing**: Automatically restart failed pods
- **Rollback**: Easily rollback to previous versions

#### Scaling Example:
```bash
# Scale voting app to 3 replicas
kubectl scale deployment voting-app-deploy --replicas=3

# Scale result app to 2 replicas
kubectl scale deployment result-app-deploy --replicas=2
```

## 🎯 How to Use the Application

1. **Cast Your Vote**:
   - Open the voting interface in your browser
   - Choose between "Cats" or "Dogs"
   - Click your preference

2. **View Results**:
   - Open the results interface in your browser
   - See real-time voting results
   - Results update automatically as votes are cast

## 🔧 Troubleshooting

### Common Issues:

#### Pods Not Starting
```bash
# Check pod status and logs
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

#### Services Not Accessible
```bash
# Check service configuration
kubectl get services
kubectl describe service <service-name>

# For Minikube, ensure tunnel is running
minikube tunnel
```

#### Database Connection Issues
```bash
# Check if PostgreSQL pod is ready
kubectl logs postgres-pod

# Verify environment variables
kubectl describe pod postgres-pod
```

## 🧹 Cleanup

To remove all deployed resources:

```bash
# Delete all pods
kubectl delete pod voting-app-pod result-app-pod worker-app-pod redis-pod postgres-pod

# Delete all services
kubectl delete service voting-service result-service redis db
```

Or delete everything at once:
```bash
kubectl delete -f .
```

## 📁 Project Structure

```
voting-app-demo/
├── README.md                    # This file
├── deployment-yaml/             # Deployment configurations
│   ├── voting-app-depoloy.yaml     # Voting app deployment
│   ├── result-app-deploy.yaml      # Result app deployment
│   ├── worker-app-deploy.yaml      # Worker deployment
│   ├── redis-deploy.yaml           # Redis deployment
│   └── postgres-deploy.yaml        # PostgreSQL deployment
├── voting-app-pod.yaml         # Voting frontend pod
├── voting-app-serice.yaml      # Voting frontend service
├── redis-pod.yaml              # Redis cache pod
├── redis-service.yaml          # Redis service
├── worker-pod.yaml             # Vote processor pod
├── postgres-pod.yaml           # PostgreSQL database pod
├── postgres-service.yaml       # PostgreSQL service
├── result-app-pod.yaml         # Results frontend pod
└── result-app-service.yaml     # Results frontend service
```

## 🔍 Understanding the Configuration

### Resource Limits
Each pod is configured with:
- **Memory**: 64Mi request, 128Mi limit
- **CPU**: 250m request, 500m limit

### Network Configuration
- **Voting App**: Accessible on port 30004
- **Result App**: Accessible on port 30005
- **Internal Services**: Redis (6379), PostgreSQL (5432)

### Database Configuration
- **PostgreSQL**:
  - Username: `postgres`
  - Password: `postgres`
  - Database: `db`

## 🎓 Learning Objectives

This demo teaches:
- Kubernetes pod and service concepts
- Microservices architecture
- Container orchestration
- Service discovery and networking
- Persistent vs temporary data storage

## 🆘 Getting Help

If you encounter issues:
1. Check the troubleshooting section above
2. Verify all prerequisites are installed correctly
3. Ensure your Kubernetes cluster is running
4. Check pod logs for specific error messages

## 📚 Next Steps

After successfully running this demo:
- Explore Kubernetes Deployments vs Pods
- Learn about ConfigMaps and Secrets
- Try scaling the application
- Implement persistent volumes for the database

---

**Note**: This is a demo application for learning purposes. For production use, implement proper security measures, persistent storage, and monitoring.