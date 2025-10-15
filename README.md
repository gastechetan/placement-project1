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
   - 
```
pipeline {
    agent any

    environment {
        # --- DOCKERHUB LOGIN DETAILS ---
        DOCKERHUB_USERNAME = "Chetan"
        DOCKERHUB_PASSWORD = "Redhat@123"

        # --- AWS ACCESS DETAILS ---
        AWS_ACCESS_KEY_ID = ""  // Add via Jenkins credentials ideally
        AWS_SECRET_ACCESS_KEY = ""
        AWS_DEFAULT_REGION = "ap-south-1"

        # --- DOCKER IMAGES ---
        BACKEND_IMAGE = "Chetan/spring-backend:v1"
        FRONTEND_IMAGE = "Chetan/angular-frontend:v1"

        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        PATH = "/usr/local/bin:/usr/bin:/bin:$PATH"
    }

    stages {

        stage('Git Clone') {
            steps {
                echo "📦 Cloning Repository..."
                git branch: 'main', url: 'https://github.com/gastechetan/placement-project1.git'
            }
        }

        stage('SonarQube Code Analysis') {
            environment {
                SONARQUBE = credentials('sonarqube-token')
            }
            steps {
                echo "🔍 Running SonarQube Code Analysis..."
                withSonarQubeEnv('SonarQube') {
                    
                    dir('spring-backend') {
                        sh '''
                        echo "📦 Running SonarQube analysis for Spring Backend..."
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=placement-backend \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONARQUBE
                        '''
                    }

                    dir('angular-frontend') {
                        sh '''
                        echo "🎨 Running SonarQube analysis for Angular Frontend..."
                        if ! command -v sonar-scanner >/dev/null 2>&1; then
                            echo "📦 Installing Sonar Scanner CLI..."
                            npm install -g sonar-scanner
                        fi

                        sonar-scanner \
                            -Dsonar.projectKey=placement-frontend \
                            -Dsonar.sources=src \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONARQUBE
                        '''
                    }
                }
            }
        }

        stage('Configure AWS CLI') {
            steps {
                sh '''
                set -e
                echo "⚙️ Configuring AWS CLI..."
                mkdir -p ~/.aws
                cat > ~/.aws/credentials <<EOF
```[default]
aws_access_key_id=${AWS_ACCESS_KEY_ID}
aws_secret_access_key=${AWS_SECRET_ACCESS_KEY}
EOF

                cat > ~/.aws/config <<EOF
[default]
region=${AWS_DEFAULT_REGION}
EOF

                echo "✅ AWS CLI configured successfully."
                '''
            }
        }

        stage('Check AWS & EKS Access') {
            steps {
                sh '''
                set -e
                echo "🔍 Validating AWS and EKS access..."
                aws sts get-caller-identity
                aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name cluster
                kubectl get nodes
                echo "✅ AWS and Kubernetes pre-check successful!"
                '''
            }
        }

        stage('Build & Push Backend Docker') {
            steps {
                dir('spring-backend') {
                    sh '''
                    set -e
                    echo "🐳 Building backend Docker image..."
                    docker build -t $BACKEND_IMAGE .
                    echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                    docker push $BACKEND_IMAGE
                    docker logout
                    echo "✅ Backend image pushed successfully!"
                    '''
                }
            }
        }

        stage('Build & Push Frontend Docker') {
            steps {
                dir('angular-frontend') {
                    sh '''
                    set -e
                    echo "🐳 Building frontend Docker image..."
                    docker build -t $FRONTEND_IMAGE . 
                    echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                    docker push $FRONTEND_IMAGE
                    docker logout
                    echo "✅ Frontend image pushed successfully!"
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('k8s') {
                    sh '''
                    set -e
                    echo "🚀 Starting Kubernetes deployment..."

                    echo "➡️ Applying Backend Deployment..."
                    kubectl apply -f backend-deployment.yaml --kubeconfig=$KUBECONFIG
                    kubectl apply -f backend-service.yaml --kubeconfig=$KUBECONFIG

                    echo "➡️ Applying Frontend Deployment..."
                    kubectl apply -f frontend-deployment.yaml --kubeconfig=$KUBECONFIG
                    kubectl apply -f frontend-service.yaml --kubeconfig=$KUBECONFIG

                    echo "➡️ Applying Ingress..."
                    kubectl apply -f ingress.yaml --kubeconfig=$KUBECONFIG

                    echo "✅ Kubernetes deployment completed successfully!"
                    '''
                }
            }
        }

        stage('Verify Services') {
            steps {
                sh '''
                echo "🔍 Verifying deployed services..."
                kubectl get pods,svc,ingress --kubeconfig=$KUBECONFIG
                echo "✅ All services verified successfully!"
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check console logs for details."
        }
    }
}

````
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
```

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
````
`````

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


