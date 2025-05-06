# node-k8s-app-Gitops
## Project Overview

You’ll build a simple Node.js + Express + MongoDB app and deploy it to:

1. Local Kubernetes (Minikube) using:

    - Helm
    - Kustomize

2. AWS EKS using:

    - Helm

    - Kustomize
    (EKS will be provisioned using cost-effective Terraform modules.)

3. GitOps deployment using:

    - ArgoCD

    Everything will follow DevOps best practices, using:

    - Modular folder structures

    - Production-ready YAML/Helm charts

    - Infrastructure as Code (Terraform)

    - GitOps workflows

## Phase 1: App Development
### Step 1: Project Structure
node-k8s-app/
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   └── index.js
│   ├── Dockerfile
│   ├── .env
│   └── package.json
├── kubernetes/
│   ├── helm/
│   └── kustomize/
├── infra/
│   ├── terraform/
│   │   └── eks/
├── gitops/
│   └── argocd/
└── README.md

### Step 2: Initialize Node + Express API
backend/package.json
```json
{
  "name": "node-k8s-app",
  "version": "1.0.0",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.2.2",
    "dotenv": "^16.3.1"
  }
}

```
backend/src/index.js (code is contained in project folder)
backend/.env (contained in .env file)

## Next Steps
#### We'll:

- Containerize the App with Docker

- Set up Minikube

- Deploy with Helm

- Deploy with Kustomize

- Provision AWS EKS with Terraform

- Deploy to EKS with Helm/Kustomize

- Apply GitOps using ArgoCD


