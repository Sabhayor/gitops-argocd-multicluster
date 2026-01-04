# Advanced GitOps with ArgoCD: Multi-Cluster, Microservices, and CI/CD Integration

## Introduction

This project demonstrates **advanced GitOps techniques using ArgoCD**, focusing on real-world deployment scenarios such as:

* Multi-cluster Kubernetes deployments
* Microservices architectures
* CI/CD pipeline integration with ArgoCD
* Automated and declarative application delivery

By completing this project, you will gain practical experience managing **complex Kubernetes environments using Git as the single source of truth**.

---

## Prerequisites

Before starting, ensure you have the following:

* Kubernetes clusters (EKS, GKE, AKS, or Minikube)
* `kubectl` installed and configured
* `argocd` CLI installed and authenticated
* Docker installed
* GitHub account
* Basic understanding of Kubernetes, Git, and CI/CD concepts

---

## Project Architecture Overview

```text
Developer → GitHub → CI Pipeline → Container Registry
                      ↓
                 GitOps Repo
                      ↓
                 ArgoCD
                      ↓
          Multiple Kubernetes Clusters
```

---

## Deploying Multi-Cluster and Microservices Architectures with ArgoCD

### Objective

Deploy applications across **multiple Kubernetes clusters** and manage **independent microservices** using ArgoCD.

---

## Step 1: Setting Up a Multi-Cluster Kubernetes Environment

### 1.1 Create Multiple Clusters

You can use **AWS EKS** (recommended for realism) or **Minikube** (local testing).

**EKS Cluster:**

```bash
eksctl create cluster --name dev-cluster --region us-east-1
eksctl create cluster --name prod-cluster --region us-east-1
```

**Minikube Cluster**
```bash
minikube start -p dev-cluster --driver=docker
minikube start -p prod-cluster --driver=docker
```
List all minikube clusters
```
minikube profile list
```

Verify contexts:

```bash
kubectl config get-contexts
```

---

### 1.2 Verify Cluster Access

Switch contexts:

```bash
kubectl config use-context dev-cluster
kubectl get nodes
```

Repeat for the production cluster.

---

## Step 2: Registering Multiple Clusters with ArgoCD

Login to ArgoCD:

```bash
argocd login <ARGOCD_SERVER>
```

Add clusters:

```bash
argocd cluster add dev-cluster
argocd cluster add prod-cluster
```

**Explanation**
This command grants ArgoCD permission to manage resources in the specified cluster context.

Verify:

```bash
argocd cluster list
```

---

## Step 3: Deploying Applications to Multiple Clusters

### 3.1 Repository Structure (Multi-Cluster)

```text
gitops-repo/
├── apps/
│   ├── dev/
│   │   └── application.yaml
│   └── prod/
│       └── application.yaml
```

---

### 3.2 Sample ArgoCD Application (Production)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-prod
  namespace: argocd
spec:
  project: default
  destination:
    server: https://kubernetes.default.svc
    namespace: prod-namespace
  source:
    repoURL: https://github.com/Sabhayor/gitops-argocd-multicluster.git
    targetRevision: main
    path: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Explanation**

* `repoURL`: GitOps repository
* `path`: Environment-specific manifests
* `automated`: Enables continuous deployment

---

## Step 4: Managing Microservices with ArgoCD

### 4.1 Microservices Repository Structure

```text
microservices-repo/
├── auth-service/
│   ├── deployment.yaml
│   └── service.yaml
├── payment-service/
│   ├── deployment.yaml
│   └── service.yaml
├── order-service/
│   ├── deployment.yaml
│   └── service.yaml
```

---

### 4.2 Separate ArgoCD Applications per Microservice

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: auth-service
  namespace: argocd
spec:
  destination:
    namespace: auth
    server: https://kubernetes.default.svc
  source:
    repoURL: https://github.com/your-org/microservices-repo.git
    path: auth-service
    targetRevision: main
  syncPolicy:
    automated:
      selfHeal: true
```

**Benefits**

* Independent deployments
* Easier rollbacks
* Fine-grained scaling
* Reduced blast radius

---

## Additional Considerations

### Inter-Cluster Communication

* Use service mesh (Istio, Linkerd)
* Use cloud-native networking (AWS VPC Peering, Load Balancers)

### Security & Compliance

* RBAC per cluster
* ArgoCD Projects for isolation
* Policy enforcement (OPA/Gatekeeper)

---

## Building and Managing a CI/CD Pipeline Using ArgoCD

### Objective

Integrate **CI pipelines** with **ArgoCD-based CD** for full GitOps automation.

---

## Step 1: Setting Up CI Pipeline (GitHub Actions)

### 1.1 Sample GitHub Actions Workflow

```yaml
name: Build and Push Image

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Build Docker Image
        run: docker build -t my-app:${{ github.sha }} .

      - name: Login to Registry
        run: docker login -u ${{ secrets.DOCKER_USER }} -p ${{ secrets.DOCKER_PASS }}

      - name: Push Image
        run: docker push my-app:${{ github.sha }}
```

---

## Step 2: Updating Kubernetes Manifests Automatically

### Example Image Update (CI Step)

```bash
sed -i "s|image: my-app:.*|image: my-app:${GITHUB_SHA}|g" deployment.yaml
git commit -am "Update image tag"
git push origin main
```

**Key Concept**
CI updates Git → ArgoCD detects change → CD happens automatically.

---

## Step 3: ArgoCD Auto-Sync Configuration

Enable auto-sync in Application spec:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

---

## Step 4: Webhook Integration

* Configure GitHub webhook to ArgoCD
* Ensures near real-time synchronization

---

## Rollback Strategy

```bash
argocd app rollback my-app <REVISION>
```

or revert Git commit:

```bash
git revert <commit-id>
git push
```

---

## Case Study Analysis

### Example: Financial Institution Using ArgoCD

**Challenges**

* Multi-cluster compliance
* Audit requirements
* Deployment consistency

**Solution**

* ArgoCD managing multiple EKS clusters
* Git-based approvals
* Automated drift detection

**Outcome**

* Faster deployments
* Improved compliance
* Reduced manual errors

---

## GitOps Best Practices

### Repository Structure

```text
gitops/
├── applications/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── shared-components/
```

---

### Secret Management

* Sealed Secrets
* SOPS + KMS
* External Secrets Operator

---

### Multi-Environment Strategy

* Use Kustomize or Helm
* Environment overlays
* Immutable image tags

---

## Recommended Resources

* ArgoCD Documentation
* GitOps Best Practices
* Kubernetes Microservices Patterns
* CNCF Case Studies

---

