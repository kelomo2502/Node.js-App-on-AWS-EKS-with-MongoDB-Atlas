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

```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

```
c. install minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version

```

3. Start Minikube
`minikube start --driver=docker`
Explanation:

- --driver=docker: Tells Minikube to use Docker as the VM/container runtime.

- If using VirtualBox or none, run minikube drivers to list what's supported.

4. Confirm It Works
`kubectl get nodes`
You should see 1 node in the Ready state, like:
```bash
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   1m    v1.29.0

```

5. Enable Ingress Controller (Important for Web Access)
Kubernetes needs an Ingress controller to expose services to your browser.
`minikube addons enable ingress`
Check its status:
`kubectl get pods -n ingress-nginx`

6. Use Minikube Docker Daemon (Optional but Helpful)
So you don’t need to push images to DockerHub:
`eval $(minikube docker-env)`

Now, when you run:
`docker build -t node-k8s-app .`
The image is built inside Minikube’s Docker environment, and Kubernetes can access it directly.

| Command              | Purpose                                    |
| -------------------- | ------------------------------------------ |
| `minikube status`    | Check status of the cluster                |
| `minikube dashboard` | Launches a visual dashboard (try it!)      |
| `minikube stop`      | Stop the cluster                           |
| `minikube delete`    | Delete the cluster                         |
| `minikube tunnel`    | Exposes LoadBalancer services to localhost |

## Troubleshooting Tips
Permission denied on Docker: Make sure you ran sudo usermod -aG docker $USER and newgrp docker.

Ingress pods stuck: Run kubectl describe pod to check errors. Usually a restart helps.

Driver issues: Run minikube start --driver=none (Linux only) or --driver=virtualbox if Docker fails.

✅ At This Point You Should Have:
A working Minikube Kubernetes cluster

Docker and kubectl installed

The ability to build Docker images inside Minikube


## Phase 3: Deploying the App to Minikube Using Helm
### What is Helm?
**Helm is the package manager for Kubernetes. Think of it like apt for Ubuntu or npm for Node.js—but for Kubernetes apps. It lets you template Kubernetes YAML files, reuse configurations, and manage deployments easily.**

## Folder Structure
We'll create this inside the project root:
project-root/
├── backend/
├── helm/
│   └── node-app/         
│       ├── templates/
│       └── values.yaml

## Step-by-Step Guide
1. Create the Helm Chart
In your project root:

```bash
mkdir -p helm
cd helm
helm create node-app

```
This creates a sample chart with default templates.

2. Understand the Helm Chart Layout
```bash
node-app/
├── Chart.yaml            # Info about the chart (name, version)
├── values.yaml           # Configurable values
├── templates/            # All Kubernetes YAML templates
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ...
```
3. Clean Up Unnecessary Files
You can remove or empty files like:
```bash
rm templates/tests/*
rm templates/ingress.yaml templates/hpa.yaml templates/serviceaccount.yaml

```
Then clean values.yaml to something simpler:
```yaml
# charts/node-app/values.yaml
replicaCount: 1

image:
  repository: node-k8s-app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 3000

mongodb:
  uri: "mongodb://your-mongodb-url"

resources: {}


```
4. Edit deployment.yaml Template
Modify templates/deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 3000
          env:
            - name: MONGO_URI
              value: {{ .Values.mongodb.uri | quote }}

```
5. Edit service.yam
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
spec:
  selector:
    app: {{ .Chart.Name }}
  ports:
    - port: 3000
      targetPort: 3000
  type: {{ .Values.service.type }}

```
6. Install the App with Helm
Ensure your image is built inside Minikube’s Docker if you don't plan to use image from a remote container registry:
```bash
eval $(minikube docker-env)
docker build -t node-k8s-app ./backend
```

Now install with Helm:
```bash
cd helm
helm install node-app ./node-app
```
Check status:
`kubectl get all`
7. Access the App
Expose it temporarily via port-forward:
`kubectl port-forward service/node-app 3000:3000`

[Then visit:](http://localhost:3000)

## Summary of Helm Benefits
Reuse templates with variables (values.yaml)

Package your entire app for sharing or CI/CD

Great for complex or production-ready deployments

## Phase 4: Deploying to Minikube Using Kustomize
### What is Kustomize?
Kustomize lets you customize raw Kubernetes YAML files using overlays and patches — no templating language or values files like Helm. You work with plain YAML and layer environments (dev, prod, etc.) cleanly.

### Folder Structure (Kustomize Style)
We’ll set it up like this:

kustomize/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patch-env.yaml

#### Step-by-Step Guide
1. Create Directory Structure
```bash
mkdir -p kustomize/base
mkdir -p kustomize/overlays/dev

```
2. Write the Base Deployment
kustomize/base/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-k8s-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-k8s-app
  template:
    metadata:
      labels:
        app: node-k8s-app
    spec:
      containers:
        - name: node-k8s-app
          image: node-k8s-app:latest
          ports:
            - containerPort: 3000
          env:
            - name: MONGO_URI
              value: "mongodb://your-mongodb-url"

```
3. Write the Base Service
kustomize/base/service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: node-k8s-app
spec:
  selector:
    app: node-k8s-app
  ports:
    - port: 3000
      targetPort: 3000
  type: ClusterIP

```
4. Create kustomization.yaml for Base
kustomize/base/kustomization.yaml
```yaml
resources:
  - deployment.yaml
  - service.yaml

```
7. Build Docker Image
Make sure you're using Minikube Docker env if you are not pulling image from remote container registry:
```bash
eval $(minikube docker-env)
docker build -t node-k8s-app ./backend

```
8. Apply with Kustomize
Navigate to overlay folder and apply:
`kubectl apply -k kustomize/overlays/dev`