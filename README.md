# DevOps Assignment — Documentation & Reporting 

This document provides a step-by-step guide, screenshots, tools description, and reflections for the Module 7 DevOps assignment: CI/CD + Kubernetes deployment of the Next.js project.

---

## 1️⃣ Step-by-Step Setup Instructions

### **EC2 Setup**
1. Launch an **Ubuntu 22.04 t3.medium+ EC2 instance**.
2. SSH into the instance:
    ssh -i "keynov.pem" ubuntu@ec2-3-145-137-5.us-east-2.compute.amazonaws.com

**Install Required Tools**

# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/

**
**Kind Cluster Setup****
1. Create kind-config.yaml:
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: ostad-cluster
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
  - containerPort: 443
    hostPort: 443
  - containerPort: 30000
    hostPort: 30000
- role: worker
- role: worker


**2. Create the cluster:**
kind create cluster --config kind-config.yaml

**
**3. Create the ostad namespace:****
kubectl create namespace ostad

**Deploy Application**
kubectl apply -f deployment.yaml -n ostad
kubectl apply -f service.yaml -n ostad

**Verification**
kubectl get nodes

**Namespace:**
kubectl get ns

**Verify Deployment**
kubectl get all -n ostad

**Verify Service and pods**
kubectl get pods -n ostad
kubectl get svc -n ostad

**Access Application**
URL: http://3.145.137.5:30000

**Tools & Commands Used**

| Tool               | Purpose                                              |
| ------------------ | ---------------------------------------------------- |
| **Docker**         | Build, run, and push the Next.js app container image |
| **kubectl**        | Manage Kubernetes resources                          |
| **Kind**           | Create local Kubernetes cluster inside Docker        |
| **GitHub Actions** | CI pipeline to build, test, and push Docker image    |



Reflection on Challenges & Solutions
| Challenge                                | Solution                                                           |
| ---------------------------------------- | ------------------------------------------------------------------ |
| `kubectl` connection refused             | Created kubeconfig for ubuntu user and set permissions correctly   |
| Kind cluster nodes not ready immediately | Added sleep/wait after Docker startup to ensure cluster creation   |
| Docker image pull failure                | Verified image exists on Docker Hub and updated deployment.yaml    |
| Application not accessible publicly      | Configured NodePort service and mapped ports in `kind-config.yaml` |


**Notes**

Replace <EC2_PUBLIC_IP> with your actual EC2 public IP when accessing the app.

Ensure Docker Hub credentials are valid for pulling the image.

All files (deployment.yaml, service.yaml, kind-config.yaml, .github/workflows/ci.yml) are included in this repository.

