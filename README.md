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

## Phase 1 (continued): Dockerizing the Node.js App
Step 1: Create the Dockerfile
Go to the backend/ folder and create a file named Dockerfile.

Here’s the complete content and detailed explanation:

```Dockerfile
# Stage 1: Build stage
FROM node:18-alpine AS builder

# Install build tools
RUN apk --no-cache --update upgrade && \
    apk add --no-cache python3 make g++

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY package-lock.json ./

# Install dependencies
RUN npm ci --ignore-scripts

# Copy all source files
COPY . .

# Stage 2: Production stage
FROM node:18-alpine

# Security updates
RUN apk --no-cache --update upgrade

WORKDIR /app

# Create non-root user
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup && \
    chown -R appuser:appgroup /app

# Copy package files
COPY --from=builder --chown=appuser:appgroup /app/package*.json ./
COPY --from=builder --chown=appuser:appgroup /app/package-lock.json ./

# Install production dependencies
RUN npm ci --production --ignore-scripts && \
    npm cache clean --force

# Copy application files from builder
COPY --from=builder --chown=appuser:appgroup /app/src ./src
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules

# Environment
ENV NODE_ENV=production PORT=3000

# Healthcheck
HEALTHCHECK --interval=30s --timeout=5s \
  CMD node -e "require('http').request({host:'localhost',port:3000},(r)=>{process.exit(r.statusCode===200?0:1)}).end()"

# Runtime
USER appuser
EXPOSE 3000

# Find and use the correct entry point (adjust if needed)
CMD ["node", "src/server.js"]

```
## Step 2: Add a .dockerignore File
Still in the backend/ folder, create a .dockerignore file to avoid copying unnecessary files into the container.
```bash
node_modules
.env

```
This:

- Skips uploading node_modules (we'll rebuild them in Docker)
- Skips the .env file (we’ll inject env vars securely using Kubernetes later)

## Step 3: Build and Run the Docker Image Locally (Optional Test)
If you want to test the image locally (before deploying to Kubernetes):

Open a terminal in the backend/ folder.

Run:

```bash
# Build the image
docker build -t node-k8s-app .

# Run the container
docker run -p 3000:3000 --env MONGO_URI="your-local-mongo-uri" node-k8s-app

```
Then visit http://localhost:3000 in your browser — it should say:
Node K8s App running!

## Phase 2: Setting Up Minikube (Local Kubernetes Cluster)
### What is Minikube?
Minikube is a lightweight Kubernetes implementation that runs a single-node cluster inside a virtual machine (or container) on your local machine. It lets you test and deploy Kubernetes apps locally.

### Step-by-Step Minikube Setup (Detailed for Beginners)
1. System Requirements
- OS: Linux, macOS, or Windows

- At least 2 CPUs, 2GB RAM, and 20GB free space

- Virtualization support (e.g., VT-x/AMD-v enabled in BIOS)

2. Install Required Tools
a. Install Docker
Docker is required to run containers, which Minikube will use.

- Ubuntu/Debian:
```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
newgrp docker

```
- Then verify `docker --version`

b. Install Kubectl
kubectl is the CLI tool to interact with Kubernetes.

Linux:

````bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

```