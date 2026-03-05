# vCluster (Vind)

## Table of Contents

1. [What is vCluster?](#what-is-vcluster)
2. [Key Features](#key-features)
3. [Prerequisites](#prerequisites)
4. [Installation](#installation)
5. [Basic Setup](#basic-setup)
6. [Multi-Node Cluster](#multi-node-cluster)
7. [Load Balancer Example](#load-balancer-example)
8. [Pause & Resume Cluster](#pause--resume-cluster)
9. [Add External Node](#add-external-node)

---

## What is vCluster?

`vind` stands for "VCluster in Docker" - a project by Loft Labs that runs virtual Kubernetes clusters as Docker containers.

**Key Idea:** Your entire Kubernetes cluster runs as a lightweight Docker container on your machine.

**vs Kind:**
- Kind: Creates local Kubernetes clusters
- vCluster: Lighter weight, more features, faster

---

## Key Features

### Pause & Resume Clusters
- Pause unused clusters to save resources
- Resume anytime without losing data
- Unlike Kind: must kill and restart cluster

### Built-in User Interface
- No need to learn kubectl
- Browse pods, logs, resources visually
- Easier for non-DevOps developers

### Load Balancer Support
- Create LoadBalancer service type (Kind doesn't support this)
- Get external IP automatically
- No need for Ingress or MetalLB

### Docker Integration
- Load images directly without pushing to registry
- Cluster runs locally, so it sees local Docker images

### Connect External Nodes
- Add external EC2 instances as worker nodes
- Scale up local cluster with cloud resources
- Example: Scale proof-of-concept with real nodes

### Performance
- Faster than Kind
- Lighter weight
- Team actively adding features (snapshots coming soon)

---

## Prerequisites

- Docker running
- kubectl installed

---

## Installation

### macOS
```bash
brew install loft-sh/tap/vcluster
```

### Linux/Windows
Check documentation for OS-specific commands
```bash
https://www.vcluster.com/install
````

### Verify Installation
```bash
vcluster version
```

---

## Basic Setup

### Set Docker as Driver
```bash
vcluster use driver docker
```

### Start UI Platform (Optional)
```bash
vcluster platform start
```
Access dashboard in browser

### Create Single Node Cluster
```bash
vcluster create test-cluster
```

After creation:
- kubectl is automatically configured to new cluster
- Cluster appears in dashboard UI

---

## Multi-Node Cluster

**Create values.yaml:**
```yaml
experimental:
  docker:
    nodes: 
    - name: "worker-1"
      ports:
        - "9090:9090"
    - name: "worker-2"
      volumes:
        - "/tmp/data:/data"
      env:
        - "NODE_ROLE=worker"
```

**Create cluster:**
```bash
vcluster create multi-node-cluster -f values.yaml
```

Result: Control plane + 2 worker nodes automatically joined

---

## Load Balancer Example

**Create service (load-balancer-svc.yaml):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
  labels:
    app: nginx
spec:
  type: LoadBalancer  #NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
```

**Apply and access:**
```bash
kubectl apply -f load-balancer-svc.yaml
kubectl get pods -w # To Watch Pod Status Continually 
kubectl get svc
# Copy EXTERNAL-IP and open in browser
```

Unlike Kind: No port-forward or tunnel needed

---

## Pause & Resume Cluster

**Pause cluster (saves resources):**
```bash
vcluster pause multi-node-cluster
```

Output: Stopping nodes, services, resources

**Resume cluster:**
```bash
vcluster resume multi-node-cluster
```

Takes 1-2 seconds - resources automatically restart in running state

---

## Add External Node

**Prerequisites:**
- EC2 instance (or any external VM)
- SSH access to instance

**Create cluster with VPN enabled:**
```bash
vcluster create super-cluster \
  --set privateNodes.vpn.enabled=true \
  --set privateNodes.vpn.nodeToNode.enabled=true
```

**Generate token on local machine:**
```bash
vcluster token create
```

**On EC2 instance - paste and run the token command:**
```bash
# Token command copied from local machine
# Automatically joins EC2 as worker node
```
**Verify:**
```bash
kubectl get nodes
kubectl describe node <node-name> # Check node details

#Verify in dashboard:
- Control plane node + EC2 worker node visible
- Can now use EC2 resources in local cluster
```

---

## Using vCluster in GitHub Actions

**Create configmap.yml**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
  namespace: default
data:
  APP_NAME: "MyApp"
  APP_ENV: "production"
  LOG_LEVEL: "info"
```

**GitHub Actions Workflow structure:**

1. Checkout repository
2. Install kubectl
3. Login to container registry (optional)
4. Install vcluster CLI
5. Set docker driver
6. Create vcluster
7. Deploy your app/config
8. Verify deployment

**GitHub workflow example (.github/workflows/ci.yml):**

```yaml
name: Deploy ConfigMap using VIND

on:
  push:
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      # ----- Checkout -----
      - name: Checkout repository
        uses: actions/checkout@v4

      # In Ubuntu Docker Comes Out of Box
      # ----- Install kubectl -----
      - name: Install kubectl
        uses: azure/setup-kubectl@v4

      # ----- Login to GHCR -----
      - name: Login to GitHub Container Registry
        run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

      # ----- Install vcluster CLI -----
      - name: Install vcluster CLI
        run: |
          curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-linux-amd64"
          sudo install -c -m 0755 vcluster /usr/local/bin
          rm -f vcluster

     # ----- Create vcluster (VIND mode) -----
      - name: Create vcluster
        run: |
          vcluster use driver docker
          vcluster create demo-cluster

     # ----- Deploy ConfigMap -----
      - name: Apply ConfigMap
        run: |
          kubectl apply -f configmap.yml

      # ----- Verify -----
      - name: Verify ConfigMap
        run: |
          kubectl get configmap
          
```

**Key points:**
- Docker comes out of box on Ubuntu
- Token generated automatically in CI
- Takes ~40 seconds to setup cluster
- Perfect for testing before production
- Completely free for open source projects

---

## Common Commands

```bash
# List all clusters
vcluster list

# Get current cluster context
kubectl config current-context

# Get cluster status
vcluster status <cluster-name>

# Connect to existing cluster
vcluster use <cluster-name>

# Delete cluster
vcluster delete <cluster-name>

# Check nodes
kubectl get nodes

# Check pods
kubectl get pods
kubectl get pods -A

# Check services
kubectl get svc

# Platform & UI Management
vcluster platform start
vcluster platform stop
vcluster platform status
vcluster platform logs

```
