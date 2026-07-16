# 🚀 End-to-End GitOps CI/CD Pipeline on AWS — Automated App Delivery with GitHub Actions, ArgoCD, Helm & EKS

> **Eliminating manual deployments and reducing release cycle time through a fully automated, audit-ready GitOps pipeline — built on AWS EKS, ArgoCD, Terraform, and GitHub Actions.**

---

## 📌 Project Overview

This project demonstrates a **production-grade GitOps CI/CD pipeline** that automates the full software delivery lifecycle — from source code commit to Kubernetes deployment on AWS. It reflects real-world DevOps practices used by engineering teams to achieve faster, safer, and more reliable releases.

The pipeline is structured across **three dedicated GitHub repositories**, each with a clear responsibility, following the GitOps principle of treating infrastructure and configuration as code:

| Repository | Purpose |
|---|---|
| `vprofile-app` | Application source code, CI pipeline, Docker build & push |
| `vprofile-infra` | Terraform code to provision AWS EKS cluster and infrastructure |
| `vprofile-helm` | Helm charts for Kubernetes manifests, managed by ArgoCD |

---

## 🏗️ Architecture Diagram

> The diagram below illustrates the complete GitOps flow — from developer commit to live deployment on EKS.

![Architecture Diagram](./screenshots/architecture-diagram.png)

<!-- 📸 Screenshot: Full architecture diagram showing GitHub → CI Pipeline → ECR → EKS → ArgoCD flow -->

---

## 🎯 Business Value & Organizational Impact

| Problem Solved | How This Pipeline Addresses It |
|---|---|
| Slow, error-prone manual deployments | Fully automated CI/CD pipeline triggered on every commit |
| Lack of deployment visibility & audit trail | Git is the single source of truth for all changes |
| Inconsistent environments across dev/staging/prod | Helm charts ensure reproducible, version-controlled deployments |
| Infrastructure drift | Terraform drift detection with Slack notifications |
| Security vulnerabilities shipped to production | SonarQube code quality gate blocks failing builds before merge |

---

## 🛠️ Tech Stack

- **Cloud:** AWS (EKS, ECR, EC2, IAM)
- **CI/CD:** GitHub Actions
- **Infrastructure as Code:** Terraform
- **Containerisation:** Docker
- **Container Registry:** Amazon ECR
- **Orchestration:** Kubernetes (AWS EKS)
- **GitOps Operator:** ArgoCD
- **Kubernetes Packaging:** Helm
- **Code Quality:** SonarQube
- **Notifications:** Slack
- **Build Tool:** Maven

---

## 📁 Repository Structure

```
vprofile-app/               ← This repository
├── src/                    # Java application source code
├── Dockerfile              # Docker image definition
├── pom.xml                 # Maven build configuration
└── .github/
    └── workflows/
        └── main.yml        # GitHub Actions CI pipeline

vprofile-infra/             ← Separate repository
├── main.tf                 # EKS cluster provisioning
├── variables.tf
└── .github/
    └── workflows/
        └── terraform.yml   # Terraform CI pipeline (Validate → Plan → Apply → Drift)

vprofile-helm/              ← Separate repository
├── charts/
│   └── vprofileapp/        # Helm chart for the application
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/      # K8s manifests (Deployment, Service, Ingress, PVC, Secret)
└── argo/
    └── app.yaml            # ArgoCD Application manifest
```

---

## 🔷 Part 1 — Repository Setup

### Step 1.1 — Create the Three GitHub Repositories

Three separate repositories were created to maintain **separation of concerns** — a GitOps best practice that allows each team (app developers, infrastructure engineers, platform engineers) to work independently without conflicts.

```
vprofile-app     → Application code & CI pipeline
vprofile-infra   → AWS infrastructure via Terraform
vprofile-helm    → Helm charts & ArgoCD configuration
```

<!-- 📸 Screenshot: GitHub showing all three repositories created -->

---

### Step 1.2 — Application Repository Structure (`vprofile-app`)

The application repository contains the Java source code, Maven build configuration, Dockerfile, and the GitHub Actions workflow file. The `.github/workflows/main.yml` is the heart of the CI pipeline.

<!-- 📸 Screenshot: vprofile-app repository file structure in GitHub -->

---

## 🔷 Part 2 — Infrastructure Setup (SonarQube, ECR & IAM)

### Step 2.1 — Launch SonarQube on AWS EC2

A **SonarQube server** was deployed on an EC2 instance to perform static code analysis and enforce quality gates before any Docker image is built. This ensures no low-quality or vulnerable code reaches production.

**EC2 Configuration:**
- Instance Type: `t2.medium` (minimum for SonarQube)
- OS: Ubuntu 22.04 LTS
- Security Group: Port `9000` open for SonarQube UI, Port `22` for SSH

```bash
# Install and start SonarQube (example setup)
sudo apt update && sudo apt install -y openjdk-17-jdk
# SonarQube runs on port 9000 by default
```

<!-- 📸 Screenshot: EC2 instance running in AWS console with SonarQube instance highlighted -->

---

### Step 2.2 — SonarQube Dashboard & Quality Gate Configuration

After launching SonarQube, a **project and quality gate** were configured. The quality gate defines the pass/fail criteria for code analysis — if the gate fails, the CI pipeline is halted and no image is built.

<!-- 📸 Screenshot: SonarQube dashboard showing project and quality gate status -->

---

### Step 2.3 — Create Amazon ECR Repository

An **Amazon Elastic Container Registry (ECR)** repository was created to store Docker images securely within AWS. ECR integrates natively with EKS, removing the need for external Docker Hub credentials.

```bash
# Create ECR repository via AWS CLI
aws ecr create-repository \
  --repository-name vprofile-app \
  --region us-east-1
```

<!-- 📸 Screenshot: ECR repository created in AWS console showing repository URI -->

---

### Step 2.4 — IAM Role & Permissions Setup

An **IAM user/role** with the minimum required permissions was created for GitHub Actions to authenticate with AWS. The principle of least privilege was applied — only ECR push permissions and EKS access were granted.

**Permissions attached:**
- `AmazonEC2ContainerRegistryFullAccess`
- `AmazonEKSClusterPolicy`

<!-- 📸 Screenshot: IAM user/role in AWS console showing attached policies -->

---

## 🔷 Part 3 — GitHub Actions CI/CD Pipeline (`vprofile-app`)

### Step 3.1 — Configure GitHub Secrets & Variables

All sensitive values (AWS credentials, SonarQube token, ECR registry URL) were stored as **GitHub Secrets** — never hardcoded in the pipeline file. Non-sensitive configuration values were stored as **GitHub Variables**.

**Secrets configured:**
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `SONAR_TOKEN`
- `SONAR_URL`
- `REGISTRY`

<!-- 📸 Screenshot: GitHub repository Settings → Secrets and Variables page showing secret names (values hidden) -->

---

### Step 3.2 — CI Pipeline Configuration (`main.yml`)

The GitHub Actions workflow automates the following stages on every push to the `main` branch:

```
1. Maven Build & Unit Test     → Compile code and run tests
2. Maven Checkstyle            → Enforce code style standards
3. SonarQube Code Analysis     → Static analysis and quality gate check
4. Docker Build                → Build application into a Docker image
5. Docker Push to ECR          → Push versioned image to Amazon ECR
6. Update Image Tag in Helm    → Auto-update Helm chart with new image tag (triggers ArgoCD sync)
```

```yaml
# .github/workflows/main.yml (abbreviated)
name: vprofile CI Pipeline

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Maven Build & Unit Test
        run: mvn test
        
      - name: Maven Checkstyle
        run: mvn checkstyle:checkstyle
        
      - name: SonarQube Analysis
        run: mvn sonar:sonar -Dsonar.host.url=${{ secrets.SONAR_URL }}
        
      - name: Docker Build & Push to ECR
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin ${{ secrets.REGISTRY }}
          docker build -t ${{ secrets.REGISTRY }}/vprofile-app:${{ github.sha }} .
          docker push ${{ secrets.REGISTRY }}/vprofile-app:${{ github.sha }}
          
      - name: Update Helm Image Tag
        run: |
          sed -i "s/imageTag:.*/imageTag: ${{ github.sha }}/" vprofile-helm/charts/vprofileapp/values.yaml
          git commit -am "CI: update image tag to ${{ github.sha }}"
          git push
```

<!-- 📸 Screenshot: .github/workflows/main.yml file open in GitHub showing the full pipeline -->

---

### Step 3.3 — Successful Pipeline Execution

The pipeline was triggered by a commit to `main` and all stages completed successfully — code passed the SonarQube quality gate, the Docker image was built, and pushed to ECR.

<!-- 📸 Screenshot: GitHub Actions showing all pipeline stages green / completed successfully -->

---

### Step 3.4 — Docker Image Pushed to Amazon ECR

After a successful pipeline run, the versioned Docker image is visible in the ECR repository, tagged with the Git commit SHA for full traceability.

<!-- 📸 Screenshot: Amazon ECR repository showing the pushed image with commit SHA tag -->

---

## 🔷 Part 4 — Infrastructure Provisioning (`vprofile-infra`)

### Step 4.1 — Terraform Pipeline for AWS EKS

The `vprofile-infra` repository contains Terraform code and its own GitHub Actions pipeline that provisions the **AWS EKS cluster** and all supporting infrastructure (VPC, subnets, node groups, security groups).

**Pipeline stages:**
```
Validate → Plan → Apply → Drift Detection → Notify (Slack) → Destroy (manual)
```

The **Drift Detection** stage runs on a schedule to compare the actual AWS state against the Terraform state file. If drift is detected, a **Slack notification** is sent to alert the team — preventing untracked manual changes from going unnoticed.

<!-- 📸 Screenshot: vprofile-infra GitHub Actions pipeline showing Validate → Plan → Apply stages green -->

---

### Step 4.2 — EKS Cluster Running on AWS

After `terraform apply` completes, the EKS cluster is live and visible in the AWS console, ready to receive workloads from ArgoCD.

<!-- 📸 Screenshot: AWS EKS cluster in the console showing Active status and node groups -->

---

## 🔷 Part 5 — GitOps Deployment (`vprofile-helm` + ArgoCD)

### Step 5.1 — Helm Chart Structure (`vprofile-helm`)

The `vprofile-helm` repository contains the **Helm chart** that packages all Kubernetes manifests for the application. ArgoCD watches this repository and automatically syncs any changes to the EKS cluster.

**Kubernetes resources managed by Helm:**
- `Ingress` — External traffic routing
- `Service` — Internal cluster networking
- `Deployment` — Application pod management & rolling updates
- `PersistentVolumeClaim` — Persistent storage
- `Secret` — Sensitive configuration (DB credentials, etc.)

<!-- 📸 Screenshot: vprofile-helm repository structure showing charts directory and values.yaml -->

---

### Step 5.2 — ArgoCD Setup on EKS

**ArgoCD** was installed on the EKS cluster and configured to watch the `vprofile-helm` repository. When the CI pipeline updates the image tag in `values.yaml` and pushes to the repo, ArgoCD detects the change and automatically syncs the new version to the cluster — **no manual `kubectl apply` needed**.

```bash
# Install ArgoCD on EKS
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

<!-- 📸 Screenshot: ArgoCD UI showing the application connected to the vprofile-helm repository -->

---

### Step 5.3 — ArgoCD Application Sync

The ArgoCD `Application` manifest (`argo/app.yaml`) defines which Git repo and path to watch, and which EKS cluster/namespace to deploy to. On every change to the Helm chart, ArgoCD performs a **self-healing sync** — ensuring the live cluster always matches the Git state.

```yaml
# argo/app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: vprofile-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/<your-org>/vprofile-helm
    targetRevision: main
    path: charts/vprofileapp
  destination:
    server: https://kubernetes.default.svc
    namespace: vprofile-app
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

<!-- 📸 Screenshot: ArgoCD UI showing application in Synced / Healthy state with all resources green -->

---

### Step 5.4 — Application Live on EKS

After ArgoCD syncs, the application pods are running on EKS and accessible via the configured Ingress endpoint.

```bash
# Verify pods running
kubectl get pods -n vprofile-app

# Get Ingress endpoint
kubectl get ingress -n vprofile-app
```

<!-- 📸 Screenshot: kubectl get pods output showing all pods Running, or ArgoCD resource tree showing healthy deployment -->

---

## 🔄 End-to-End Flow Summary

```
Developer pushes code to vprofile-app (main branch)
        ↓
GitHub Actions CI Pipeline triggers
        ↓
Maven Build → Checkstyle → SonarQube Analysis (Quality Gate check)
        ↓
Docker Build → Docker Push to Amazon ECR (tagged with commit SHA)
        ↓
Pipeline updates image tag in vprofile-helm values.yaml → Commits & Pushes
        ↓
ArgoCD detects change in vprofile-helm repository
        ↓
ArgoCD syncs new Helm chart to EKS cluster
        ↓
Rolling update deployed → Application live with zero downtime
```

---

## ✅ Key DevOps Practices Demonstrated

- **GitOps** — Git as the single source of truth for application and infrastructure state
- **Shift-Left Security** — Code quality and vulnerability checks before build (SonarQube)
- **Immutable Artifacts** — Docker images tagged with Git SHA, never mutated after build
- **Infrastructure as Code** — All AWS resources provisioned and version-controlled via Terraform
- **Drift Detection** — Automated alerting when infrastructure deviates from declared state
- **Self-Healing Deployments** — ArgoCD automatically corrects manual cluster changes
- **Separation of Concerns** — App code, infrastructure, and deployment config in separate repos
- **Principle of Least Privilege** — IAM scoped to minimum required permissions

---

## 📬 Connect With Me

If you found this project useful or have questions about the implementation, feel free to connect:

- 💼 [LinkedIn](https://linkedin.com/in/your-profile)
- 🐙 [GitHub](https://github.com/your-username)

---

*Built as part of a hands-on DevOps engineering portfolio to demonstrate real-world cloud-native delivery practices.*
