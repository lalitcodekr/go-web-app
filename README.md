# Go Web Application — Cloud-Native DevOps Project

> **End-to-End GitOps Deployment of a Go Web Application on Amazon EKS using ArgoCD, Helm, Docker, Kubernetes, NGINX Ingress Controller, and AWS Load Balancer.**

---

## 🖥️ Application Preview

![Website](static/images/golang-website.png)

---

## 🏗️ Architecture Overview

```
Developer
    │
    ▼
GitHub Repository
    │
    ▼
GitHub Actions (CI/CD)
    │
    ▼
Docker Hub
    │
    ▼
ArgoCD (GitOps)
    │
    ▼
Amazon EKS Cluster
    │
    ▼
Deployment + Service
    │
    ▼
NGINX Ingress Controller
    │
    ▼
AWS Elastic Load Balancer (ELB)
    │
    ▼
Internet Users
```

---

## 🚀 Quick Start (Run Locally)

```bash
go run main.go
```

The server starts on port **8080**. Access the app at:

```
http://localhost:8080/courses
```

---

## 🐳 Docker

### Build the Image

```bash
docker build -t gowebapp .
```

### Run Locally

```bash
docker run -p 8080:8080 gowebapp
```

### Push to Docker Hub

```bash
docker tag gowebapp lalitkr2526/gowebapp:v1
docker push lalitkr2526/gowebapp:v1
```

---

## ☸️ Kubernetes Manifests

All manifests are located in [`k8s/manifests/`](k8s/manifests/).

| File | Purpose |
|---|---|
| `deployment.yaml` | Deploys the Go app with replica management |
| `service.yaml` | Exposes the app inside the cluster (ClusterIP) |
| `ingress.yaml` | Routes external traffic via NGINX Ingress |

### Apply Manifests

```bash
kubectl apply -f k8s/manifests/
```

### Verify

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

---

## 📦 Helm Chart

The Helm chart is located in [`helm/gowebapp-chart/`](helm/gowebapp-chart/).

### Install via Helm

```bash
helm install gowebapp ./helm/gowebapp-chart
```

### Upgrade

```bash
helm upgrade gowebapp ./helm/gowebapp-chart
```

---

## 🔄 GitOps with ArgoCD

ArgoCD continuously watches this repository and syncs the Kubernetes cluster to match the desired state defined in `helm/gowebapp-chart/`.

### Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Apply ArgoCD Application

```bash
kubectl apply -f argocd/application.yaml
```

### GitOps Flow

```
Developer pushes code
       ↓
GitHub Actions builds & pushes Docker image
       ↓
CI updates Helm chart tag in values.yaml
       ↓
ArgoCD detects change in Git
       ↓
ArgoCD syncs cluster to desired state
       ↓
New pod deployed automatically
```

---

## 🌐 NGINX Ingress Controller

### Install

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```

### Verify

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

The NGINX Ingress Controller provisions an **AWS Elastic Load Balancer (ELB)** automatically when the service type is `LoadBalancer`.

---

## 🛠️ Amazon EKS Setup

### Prerequisites

- AWS CLI
- `kubectl`
- `eksctl`

### Create EKS Cluster

```bash
eksctl create cluster \
  --name demo-cluster \
  --region ap-south-2
```

### Configure kubectl

```bash
aws eks update-kubeconfig --region ap-south-2 --name demo-cluster
```

### Verify Nodes

```bash
kubectl get nodes
```

---

## ⚙️ CI/CD Pipeline (GitHub Actions)

The pipeline is defined in [`.github/workflows/ci.yaml`](.github/workflows/ci.yaml).

| Stage | What it does |
|---|---|
| **build** | Compiles the Go app and runs tests |
| **code-quality** | Runs `golangci-lint` for static analysis |
| **push** | Builds Docker image and pushes to Docker Hub |
| **update-newtag-in-helm-chart** | Updates the image tag in `values.yaml` and commits back to Git |

### Required GitHub Secrets

| Secret | Description |
|---|---|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `TOKEN` | GitHub Personal Access Token (for pushing to repo) |

---

## 🐞 Troubleshooting

### Common Commands

```bash
# Describe a pod (check scheduling errors)
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Check all events
kubectl get events --sort-by='.lastTimestamp'

# Check node capacity
kubectl get nodes -o jsonpath='{.items[*].status.allocatable.pods}'
```

### Known Issues & Fixes

| Issue | Root Cause | Fix |
|---|---|---|
| `ArgoCD Sync Status: Unknown` | Insufficient cluster capacity | Scale nodegroup to 3–4 nodes |
| `Too many pods` | `t3.micro` nodes support only ~4 pods each | Use larger instance type or enable prefix delegation |
| `failed calling webhook` | Broken ingress admission service | `kubectl delete validatingwebhookconfiguration ingress-nginx-admission` |
| `secret ingress-nginx-admission not found` | Admission jobs pending due to no pod slots | Scale down ArgoCD temporarily to free pod capacity |

---

## 📁 Project Structure

```
go-web-app/
├── .github/
│   └── workflows/
│       └── ci.yaml          # GitHub Actions CI/CD pipeline
├── helm/
│   └── gowebapp-chart/      # Helm chart for Kubernetes deployment
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
├── k8s/
│   └── manifests/           # Raw Kubernetes manifests
│       ├── deployment.yaml
│       ├── service.yaml
│       └── ingress.yaml
├── static/                  # HTML pages served by the app
├── Dockerfile               # Multi-stage Docker build
├── main.go                  # Go web server
├── main_test.go             # Go tests
└── go.mod
```

---

## 🧰 Tech Stack

| Category | Tools |
|---|---|
| **Language** | Go (Golang) |
| **Containerization** | Docker, Docker Hub |
| **Orchestration** | Kubernetes (Amazon EKS) |
| **Package Manager** | Helm |
| **GitOps** | ArgoCD |
| **Ingress** | NGINX Ingress Controller |
| **Cloud** | AWS (EKS, EC2, ELB) |
| **CI/CD** | GitHub Actions |
| **Infrastructure** | eksctl, AWS CLI, kubectl |

---

## 📚 Key Concepts Demonstrated

- **Containerization** — Packaging app + runtime into a portable Docker image using a multi-stage build (distroless final image)
- **Kubernetes Deployments** — Declarative desired-state management with ReplicaSets and self-healing
- **Helm** — Templated, parameterized Kubernetes packaging (the `apt` of Kubernetes)
- **GitOps** — Git as the single source of truth; ArgoCD automatically reconciles cluster state
- **NGINX Ingress** — URL/path/host routing, SSL termination, and AWS ELB provisioning
- **VPC CNI Pod Limits** — AWS networking constraints on pod density per node type
- **Production Troubleshooting** — Scheduling failures, admission webhooks, CNI limits, and capacity planning

---

## 👤 Author

**Lalit Kumar** — [GitHub](https://github.com/lalitkr2526) | [Docker Hub](https://hub.docker.com/u/lalitkr2526)

---

## 📄 License

This project is licensed under the [Apache 2.0 License](LICENSE).
