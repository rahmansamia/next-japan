# DevOps Assignment — CI/CD + Kubernetes Deployment (Next.js Project)

This repository contains the solution for the Module 7 DevOps assignment: building a Next.js application, setting up a CI/CD pipeline, deploying it to a Kubernetes cluster on AWS EC2, and exposing it publicly.

---

## 🔗 Project Repository

[Next.js Project Repository](https://github.com/mdarifahammedreza/next-japan.git)

Docker Image: `samiarahmanliza/next-japan:latest`

---

## 🎯 Objective

- Build and deploy a Next.js application using a CI pipeline.
- Create and configure a Kubernetes cluster using Kind on an AWS EC2 instance.
- Deploy the application in Kubernetes inside a namespace named `ostad`.
- Make the application publicly accessible via NodePort.

---

## Part 1 — CI Pipeline Setup 
### CI Tool Used

- GitHub Actions

### CI Pipeline Steps

1. **Install Dependencies & Build**  
  
   npm install
   npm run build

**2. Run Tests & Lint**
npm test
npm run lint

**3. Build Docker Image**
docker build -t samiarahmanliza/next-japan:latest .

**4. Push Docker Image to Docker Hub**
docker push samiarahmanliza/next-japan:latest

CI Workflow File**
.github/workflows/ci.yml



**Part 2 — EC2 Setup & Kubernetes Cluster 
EC2 Instance Details**
Instance Type: t2.medium

OS: Ubuntu 22.04

Key Pair: your-keypair-name

Installed Tools
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io curl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/

**Kind Cluster Creation**
**Kind Config (kind-config.yaml)**

yaml

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

**Command to create cluster**
kind create cluster --config kind-config.yaml

**Create Namespace**
kubectl create namespace ostad

**Verification**
kubectl get nodes

**Namespace:**
kubectl get ns

****Part 3 — Kubernetes Deployment ****
**Deployment (deployment.yaml)**
yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: next-japan-app
  namespace: ostad
  labels:
    app: next-japan
spec:
  replicas: 2
  selector:
    matchLabels:
      app: next-japan
  template:
    metadata:
      labels:
        app: next-japan
    spec:
      containers:
      - name: next-japan
        image: samiarahmanliza/next-japan:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
      restartPolicy: Always

**Service (service.yaml)**
yaml
apiVersion: v1
kind: Service
metadata:
  name: next-japan-service
  namespace: ostad
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 3000
    nodePort: 30000
  selector:
    app: next-japan


**Apply Manifests**
kubectl apply -f deployment.yaml -n ostad
kubectl apply -f service.yaml -n ostad

**Verify Deployment**
kubectl get all -n ostad

**Access Application**
URL: http://3.145.137.5:30000

