# 🚀 End-to-End GitOps CI/CD Pipeline on AWS
AWS | Terraform | Kubernetes | Docker | GitHub Actions | ArgoCD | Helm | SonarQube
### Automated Application Delivery with GitHub Actions, Terraform, Amazon EKS, Helm & ArgoCD

> A production-style GitOps CI/CD pipeline that automates the complete software delivery lifecycle—from code commit to Kubernetes deployment on Amazon EKS—using modern DevOps practices.

---

# 📌 Overview

This project demonstrates the implementation of a **production-ready GitOps pipeline** that automates application delivery while ensuring consistency, traceability, and reliability across deployments.

The solution follows GitOps principles by separating application code, infrastructure, and Kubernetes configuration into dedicated repositories. Every infrastructure and deployment change is version-controlled, enabling automated delivery with minimal manual intervention.

---

## Source Repositories

- 📦 **Application:** [vprofile-app](https://github.com/Charleyung1/vprofile-app)
- ☁️ **Infrastructure:** [vprofile-infra](https://github.com/Charleyung1/vprofile-infra)
- 🚀 **GitOps Deployment:** [vprofile-helm](https://github.com/Charleyung1/vprofile-helm)

---


# 🏗️ Architecture

The architecture below illustrates the complete deployment workflow from source code to production.

![Architecture Diagram](architecture/architecture-diagram.png)

---

# ⭐ Architecture Highlights

- Automated CI/CD pipeline using GitHub Actions
- Infrastructure provisioned with Terraform
- Kubernetes workloads deployed on Amazon EKS
- GitOps deployment using ArgoCD
- Docker images stored in Amazon ECR
- Helm used for Kubernetes package management
- Static code analysis with SonarQube
- Slack notifications for infrastructure pipeline events
- Immutable Docker image versioning using Git commit SHA
- Fully automated deployment with zero manual Kubernetes changes

---

# 💼 Business Value

| Challenge | Solution |
|------------|----------|
| Manual deployments | Fully automated CI/CD pipeline |
| Environment inconsistency | Helm-based Kubernetes deployments |
| Infrastructure drift | Automated Terraform drift detection |
| Lack of deployment visibility | Git as the single source of truth |
| Code quality risks | SonarQube Quality Gate validation |

---

# 🛠️ Technology Stack

| Category | Technologies |
|-----------|--------------|
| Cloud | AWS (EKS, ECR, EC2, IAM) |
| CI/CD | GitHub Actions |
| Infrastructure as Code | Terraform |
| Containerization | Docker |
| Orchestration | Kubernetes (Amazon EKS) |
| GitOps | ArgoCD |
| Package Management | Helm |
| Code Quality | SonarQube |
| Notifications | Slack |
| Build Tool | Maven |

---



# 📂 Repository Responsibilities

The project is intentionally divided into separate repositories to follow GitOps best practices by separating application code, infrastructure, and deployment configuration. This modular approach simplifies maintenance, enables independent versioning, and mirrors production engineering workflows.

| Repository | Responsibility |
|------------|----------------|
| **vprofile-app** | Java application source code, Dockerfile, Maven build, GitHub Actions CI pipeline, SonarQube analysis, Amazon ECR image publishing. |
| **vprofile-infra** | Terraform code provisioning AWS networking, IAM roles, Amazon EKS cluster, managed node groups, and GitHub Actions infrastructure automation. |
| **vprofile-helm** | Helm charts, Kubernetes manifests, ArgoCD Application configuration, and GitOps deployment automation. |

![Repo](screenshot/github-repositories.png)



---

# ⚙️ Continuous Integration (vprofile-app)

The application repository contains the complete Continuous Integration workflow.

Every push to the **main** branch automatically triggers the pipeline, which performs:

- Maven Build
- Unit Testing
- Checkstyle Validation
- SonarQube Code Analysis
- Docker Image Build
- Push Docker Image to Amazon ECR
- Update Helm Image Tag
- Trigger GitOps Deployment

---

## Code Quality Validation

Before a Docker image is built, the pipeline performs static code analysis using SonarQube to evaluate code quality, maintainability, and potential security issues. The configured Quality Gate prevents the pipeline from continuing if predefined standards are not met, ensuring only validated code progresses to the containerization stage.

> 📷 **Screenshot 3 – SonarQube Project Analysis with Quality Gate Passed**

![Sonar](screenshot/sonarqube-dashboard.png)


---

## GitHub Secrets & Variables

Sensitive credentials are securely managed using GitHub Secrets to prevent exposure in source code and CI/CD workflows.

The configured secrets include:

- AWS Access Key ID
- AWS Secret Access Key
- SonarQube Token
- GitOps Personal Access Token
- Helm Repository User
- Slack Webhook URL

> 📷 **Screenshot 4 – GitHub Secrets & Variables**


![github secret](screenshot/github-secrets.png)


---

## Successful CI Pipeline

After validation, GitHub Actions successfully builds and publishes the application container image.

> 📷 **Screenshot 5 – Successful GitHub Actions CI Pipeline Execution**


![github action](screenshot/github-actions-success.png)


---

## Docker Image Published to Amazon ECR

Each successful build produces a versioned Docker image tagged with the Git commit SHA for full traceability.

> 📷 **Screenshot 6 – Amazon ECR Image**


![ecr image](screenshot/ecr-image.png)


---

# ☁️ Infrastructure as Code (vprofile-infra)

Infrastructure provisioning is fully automated using Terraform.

Resources include:

- Amazon VPC
- Public Subnets
- Internet Gateway
- Route Tables
- IAM Roles
- Amazon EKS Cluster
- Managed Node Groups

GitHub Actions automates:

- Terraform Validate
- Terraform Plan
- Terraform Apply
- Drift Detection
- Slack Notifications

---

## Amazon EKS Cluster

The infrastructure repository provisions the complete Kubernetes environment using Terraform. This includes the VPC, networking components, IAM roles, managed node groups, and the Amazon EKS control plane. Automating infrastructure provisioning ensures consistency, repeatability, and eliminates manual configuration across environments.

> 📷 **Screenshot 7 – Amazon EKS Cluster**


![eks cluster](screenshot/eks-cluster.png)


---

## Slack Notifications

After the Terraform infrastructure workflow completes successfully, GitHub Actions automatically sends a notification to the configured Slack channel. The notification includes the workflow status, repository name, branch, triggering user, and a link to the workflow run, enabling the team to monitor infrastructure provisioning in real time.


> 📷 **Screenshot 8 – Slack Notification**


![slack](screenshot/slack-notification.png)



# 🚀 GitOps Deployment (vprofile-helm)

The deployment repository contains the Helm chart used to package Kubernetes resources.

ArgoCD continuously monitors the repository and automatically synchronizes any changes to Amazon EKS.

Deployment includes:

- Deployments
- Services
- Ingress
- Secrets
- Persistent Volume Claims

No manual **kubectl apply** commands are required.

---

## ArgoCD Synchronization

ArgoCD implements the GitOps deployment model by continuously monitoring the Helm repository for configuration changes. Whenever the CI pipeline updates the application image tag, ArgoCD detects the commit, compares the desired state stored in Git with the live Kubernetes cluster, and automatically performs a rolling deployment. Self-healing and automatic pruning ensure the cluster always remains synchronized with the repository.


> 📷 **Screenshot 9 – ArgoCD Healthy & Synced**


![argocd](screenshot/argocd-sync.png)


---

# 🌐 Application Deployment

After ArgoCD synchronization, the application is successfully deployed and accessible through Kubernetes Ingress.

> 📷 **Screenshot 10 – Running Application**

 



---

# 🔄 Deployment Workflow

```
Developer Commit
        │
        ▼
GitHub Actions
        │
        ▼
Build & Test
        │
        ▼
SonarQube Analysis
        │
        ▼
Docker Build
        │
        ▼
Push Image to Amazon ECR
        │
        ▼
Update Helm Chart
        │
        ▼
ArgoCD Detects Change
        │
        ▼
Deploy to Amazon EKS
        │
        ▼
Application Available
```

---

## 📊 Project Metrics

| Metric | Value |
|--------|------:|
| GitHub Repositories | 3 |
| AWS Services Used | 6 |
| CI/CD Pipelines | 2 |
| Terraform Resources | 20+ |
| Kubernetes Cluster | Amazon EKS |
| Deployment Strategy | GitOps |

---


# 📈 Results

This project successfully demonstrates:

- Production-style GitOps deployment workflow
- Fully automated CI/CD pipeline
- Infrastructure managed entirely with Terraform
- Immutable Docker image versioning
- Continuous deployment using ArgoCD
- Automated code quality validation
- Slack deployment notifications
- Self-healing Kubernetes deployments
- Zero manual production deployments

---

# ⚠️ Engineering Challenges & Solutions

Throughout the implementation of this project, several real-world challenges were encountered and resolved. These issues helped strengthen the reliability of the deployment pipeline and provided valuable troubleshooting experience.

| Challenge | Resolution |
|-----------|------------|
| **GitHub Actions failed to update the Helm repository after building and pushing the Docker image.** The workflow could not locate the correct `values.yaml` file because the Helm repository was checked out into a different directory than expected. | Updated the workflow to reference the correct repository path, verified the working directory during pipeline execution, and configured Git authentication correctly. Once resolved, the pipeline successfully updated the Helm chart with the new Docker image tag and triggered the GitOps deployment automatically. |
| **Terraform failed during Amazon EKS provisioning due to AWS IAM permission and infrastructure configuration issues.** This prevented successful creation of the EKS resources. | Reviewed the Terraform execution logs, corrected the IAM permissions required for provisioning Amazon EKS resources, validated the Terraform configuration, and re-applied the infrastructure successfully. The EKS cluster and managed node groups were created without further issues. |

### Key Takeaways

These challenges reinforced several important DevOps practices:

- Always validate repository paths when working with multiple Git repositories in CI/CD pipelines.
- Apply the principle of least privilege while ensuring IAM policies include all permissions required for infrastructure provisioning.
- Use pipeline logs and Terraform output to troubleshoot failures methodically.
- Automate deployments only after validating infrastructure and application configuration.

# ✅ DevOps Practices Demonstrated

- GitOps
- Continuous Integration
- Continuous Delivery
- Infrastructure as Code
- Immutable Infrastructure
- Kubernetes Orchestration
- Containerization
- Automated Quality Gates
- Infrastructure Drift Detection
- Least Privilege IAM
- Version-Controlled Infrastructure
- Automated Rollout Strategy

---

# 📬 Connect With Me

- 💼 LinkedIn: https://www.linkedin.com/in/charles-ekairia-44797821b/
- 💻 GitHub: https://github.com/Charleyung1

---

## ⭐ Project Summary

This project demonstrates how modern DevOps teams automate software delivery using GitHub Actions, Terraform, Amazon EKS, Docker, Helm, ArgoCD, SonarQube, and Slack. It showcases production-oriented GitOps practices for building secure, scalable, and reliable cloud-native deployment pipelines.
