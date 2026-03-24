#  End-to-End DevOps CI/CD Pipeline with GitOps (FastAPI + AWS EKS)

## Overview

This project demonstrates a complete **end-to-end DevOps pipeline** implementing:

* Continuous Integration (CI)
* Continuous Delivery (CD)
* Security Scanning
* GitOps Deployment using Argo CD
* Kubernetes-based application deployment on AWS EKS

The pipeline automates everything from **code commit → production deployment**.

---

##  Architecture

```text
Developer → GitHub (Source Repo)
        ↓
     Jenkins (CI Pipeline)
        ↓
 SonarQube (Code Quality)
        ↓
   Docker Build
        ↓
   Trivy Scan (Security)
        ↓
   AWS ECR (Image Registry)
        ↓
 GitOps Repo Update (deployment.yaml)
        ↓
     Argo CD
        ↓
   AWS EKS Cluster
        ↓
   FastAPI Application 🚀
```

---

## Tech Stack

###  Core Technologies

* Python (FastAPI)
* Docker
* Kubernetes (EKS)

### CI/CD

* Jenkins (Declarative Pipeline)

### Code Quality & Security

* SonarQube
* Trivy

###  Cloud & Infrastructure

* AWS EC2 (Jenkins, SonarQube)
* AWS ECR (Container Registry)
* AWS EKS (Kubernetes Cluster)

### GitOps

* Argo CD

---

##  Repository Structure

### 🔹 Source Code Repository

```
├── Jenkinsfile
├── Dockerfile
├── main.py
├── form.html
├── requirements.txt
├── sonar-project.properties
```

### 🔹 Deployment Repository (GitOps)

```
└── k8s/
    ├── deployment.yaml
    ├── service.yaml
```

---

##  CI/CD Pipeline Stages

### 1️Code Checkout

* Jenkins pulls source code from GitHub

### 2️Code Quality Analysis

* SonarQube scans code for bugs and vulnerabilities

### 3️Docker Build

* Application container image is built

### 4️Security Scan

* Trivy scans image for vulnerabilities

### 5️Push to ECR

* Docker image is pushed to AWS ECR

### 6️GitOps Update

* Jenkins updates `deployment.yaml` with new image tag
* Changes are committed to Deployment Repo

### 7️Continuous Deployment

* Argo CD detects changes
* Automatically syncs and deploys to EKS

*  Install Argo CD in EKS
Create a namespace:

kubectl create namespace argocd
Use this values.yaml for exposing argocd in NodePort

# Basic Argo CD values for NodePort exposure
server:
  service:
    type: NodePort
    nodePortHttp: 30080        # optional — choose a fixed NodePort (HTTP)
    nodePortHttps: 30443       # optional — choose a fixed NodePort (HTTPS)
    ports:
      http: 80
      https: 443
  ingress:
    enabled: false
  insecure: true                # serve UI without enforcing HTTPS (for lab use)
  extraArgs:
    - --insecure                # disable TLS redirection in ArgoCD server

configs:
  cm:
    # Disable TLS redirect so UI works over plain HTTP
    server.insecure: "true"

redis:
  enabled: true

controller:
  replicas: 1
repoServer:
  replicas: 1
applicationSet:
  replicas: 1
Install Argo CD using the official Helm:

# Add and update the Argo Helm repo
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install using the NodePort values file
helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  -f values.yaml
Verify installation:

kubectl get pods -n argocd
 . Access the Argo CD UI
Access via browser: Allow ==30080== port in Worker Node Security Group

https://<ec2-publicip>:30080
Get the initial admin password:

kubectl -n argocd get secret argocd-initial-admin-secret -o yaml
echo <base64secret> | base64 --decode
Login using:

Username: admin
Password: <decoded-password>

---

## Kubernetes Deployment

### Deployment

* FastAPI app deployed as Kubernetes Deployment
* Configured with resource limits and replicas

### Service

* Exposed using NodePort (for simplicity)

---

## Security Practices

* Static code analysis via SonarQube
* Container vulnerability scanning via Trivy
* No hardcoded credentials (Jenkins Credentials used)
* IAM roles used for AWS access

---

## Key Features

* Fully automated CI/CD pipeline
* Integrated security scanning
* Containerized application
* Cloud-native deployment (AWS EKS)
* GitOps-based deployment (Argo CD)
* Dynamic Docker image tagging

---

## Limitations

* Uses minimal instance types (t3.micro) for learning purposes
* NodePort used instead of LoadBalancer (cost optimization)
* No autoscaling configured

---

##  Future Improvements

* Add Ingress with domain & HTTPS
* Implement Horizontal Pod Autoscaler (HPA)
* Add monitoring (Prometheus + Grafana)
* Improve security with IAM roles for service accounts (IRSA)
* Add rollback strategy

---

##  How to Run

### 1. Push Code

Push changes to source repository

### 2. Trigger Pipeline

Jenkins automatically runs pipeline

### 3. Monitor

* Jenkins → Build status
* SonarQube → Code quality
* Argo CD → Deployment sync
* Kubernetes → Pod status

---

## Output

* Application deployed on Kubernetes
* Accessible via NodePort
* Fully automated pipeline execution

---

## Author
Dhanush N
DevOps Engineer (Learning → Building → Scaling)

---

## Conclusion

This project demonstrates a **production-style DevOps workflow** combining CI/CD, security, and GitOps principles using modern cloud-native tools.

It reflects strong foundational knowledge in:

* CI/CD automation
* Containerization
* Kubernetes
* Cloud deployment
* DevOps best practices
