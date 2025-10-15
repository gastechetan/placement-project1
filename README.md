# 🚀 DevOps End-to-End Project Documentation

## 🧩 Project Overview
This project demonstrates a complete **DevOps CI/CD pipeline** for deploying a 3-tier application (Frontend + Backend + Database) on **AWS EKS (Elastic Kubernetes Service)**.  
It automates infrastructure provisioning, continuous integration, deployment, and monitoring.

---

## ⚙️ Tech Stack

| Layer | Tools / Services |
|-------|------------------|
| Infrastructure as Code | Terraform |
| Configuration Management | Ansible |
| CI/CD Automation | Jenkins |
| Containerization | Docker |
| Orchestration | Kubernetes (EKS) |
| Monitoring | Datadog / CloudWatch |
| Code Quality | SonarQube |
| Version Control | Git & GitHub |

---

## 🏗️ 1. Infrastructure Setup (Terraform)

### Objective
Provision AWS resources for deployment — isolated, scalable, and secure.

### Steps
1. **Create VPC**
   - Custom VPC with public & private subnets.
   - Internet Gateway, NAT Gateway, and Security Groups configured.

2. **Launch EC2 Instances**
   - Jenkins, Ansible, and SonarQube servers.
   - IAM roles for secure AWS access.

3. **Create EKS Cluster**
   - Using module `terraform-aws-modules/eks/aws`.
   - Node groups configured for scalability.

4. **Networking**
   - Subnets, route tables, and security groups linked.
   - EKS connected with EC2 worker nodes.

✅ **Output:** Fully functional EKS cluster with networking and security setup.

---

## 🤖 2. Configuration Management (Ansible)

### Purpose
Automate installation and configuration of DevOps tools on EC2 instances.

### Installed Tools
- Docker  
- Kubectl  
- Terraform  
- Jenkins  
- Java (for Jenkins & SonarQube)  
- Datadog Agent (for monitoring)

---

## 🔁 3. CI/CD Pipeline (Jenkins)

### Jenkins Pipeline Stages (Declarative Pipeline)

1. **Checkout Code**
   - Pull code from GitHub repository.

2. **Code Quality Check**
   - Integrate **SonarQube** for static code analysis.
   - Enforce quality gates before build.

3. **Build & Dockerize**
   - Build frontend and backend code.
   - Create Docker images.

4. **Push to Docker Hub**
   - Tag and push Docker images to Docker Hub repository.

5. **Deploy to Kubernetes (EKS)**
   - Apply Kubernetes YAML files (deployment, service, ingress).
   - Deploy frontend, backend, and database containers.

6. **Ingress Controller**
   - Manage routing and load balancing for services.

7. **Scaling**
   - Define replica sets for auto-scaling based on resource metrics.

---

## ☁️ 4. Monitoring and Logging

### Tools Used
- **Datadog / CloudWatch / Prometheus + Grafana**

### Purpose
- Monitor cluster and node health.
- Capture application logs and metrics.
- Create dashboards for CPU, memory, and network usage.
- Set alerts for real-time anomaly detection.

---

# 🚀 Placement Project – DevOps Automation (Terraform + Jenkins + Docker + EKS)

## 📘 Overview

This project demonstrates a **complete CI/CD DevOps pipeline** that deploys an **Angular Frontend**, **Spring Boot Backend**, and **MySQL Database** on **AWS EKS (Elastic Kubernetes Service)** using **Terraform**, **Jenkins**, and **Docker**.

It automates infrastructure creation, builds Docker images, runs tests, deploys applications to Kubernetes, and integrates monitoring tools (like **Datadog** or **CloudWatch**).

---

## 🧱 Project Structure

```bash
placement-project-main/
├── Jenkinsfile                    # Jenkins Declarative Pipeline
│
├── angular-frontend/              # Angular Frontend Application
│   ├── Dockerfile                 # Dockerfile for building Angular app
│   ├── angular.json
│   ├── package.json
│   ├── package-lock.json
│   ├── karma.conf.js
│   ├── src/                       # Angular source code
│   ├── tsconfig.json
│   └── tsconfig.app.json
│
├── spring-backend/                # Spring Boot Backend Application
│   ├── Dockerfile                 # Dockerfile for backend image
│   ├── pom.xml                    # Maven dependencies and build config
│   ├── mvnw / mvnw.cmd            # Maven wrapper scripts
│   └── src/                       # Java source code
│
├── springbackend.sql              # Database schema / initialization script
│
├── k8s/                           # Kubernetes Deployment Manifests
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   └── ingress.yaml
│
└── terraform/                     # Infrastructure as Code (IaC)
    ├── main.tf                    # Root Terraform configuration
    ├── vpc/                       # VPC, Subnets, Gateways, Routes
    ├── ec2/                       # EC2 instance for Jenkins & Ansible
    ├── eks/                       # EKS cluster and node groups
    ├── rds/                       # RDS MySQL instance
    ├── ansible/                   # Ansible playbooks for configuration
    ├── main-key / main-key.pub    # SSH key pair
```

## 🔄 6. Workflow Summary

**Terraform** → creates infrastructure  
**Ansible** → configures EC2 & installs tools  
**Jenkins** → automates CI/CD pipeline  
**SonarQube** → performs code analysis  
**Docker** → builds and pushes images  
**EKS** → deploys containers  
**Datadog/CloudWatch** → monitors performance

---

## 🌟 7. Benefits

- Fully automated infrastructure provisioning & deployment.  
- Continuous Integration & Continuous Deployment enabled.  
- Scalable & secure cloud architecture.  
- Continuous feedback loop via monitoring & quality checks.  
- Easy rollback and version tracking using Git and Docker.

---

## 🧠 8. Future Enhancements
- Implement Blue/Green or Canary deployments.
- Add Helm for Kubernetes package management.
- Integrate Slack or Teams for build notifications.


