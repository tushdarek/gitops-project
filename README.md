# GitOps-Based Kubernetes Application Deployment using ArgoCD

## Overview
This project demonstrates a GitOps-driven deployment model for Kubernetes using ArgoCD.
The objective is to implement a fully automated CI/CD workflow where Git acts as the single source of truth for infrastructure and application deployments.

In this architecture, application configurations and Kubernetes manifests are stored in a Git repository. ArgoCD continuously monitors the repository and automatically synchronizes the Kubernetes cluster whenever changes are committed.

This approach eliminates manual changes in production environments and ensures consistent, traceable, and automated deployments.

---

## Key Features
* GitOps-based continuous deployment
* Kubernetes application deployment automation
* Automatic synchronization using ArgoCD
* Containerized application using Docker
* Infrastructure configuration managed through Git
* Zero manual kubectl operations in production

---
## Architecture
```
Developer
   │
   │ Code Commit
   ▼
GitHub Repository
(Source of Truth)
   │
   │ Monitored by
   ▼
ArgoCD
   │
   │ Auto Sync
   ▼
Kubernetes Cluster
   │
   ├── Deployment
   ├── Service
   │
   ▼
Application Accessible to End Users
```
---
## Technology Stack
<table border="1">
  <tr>
    <th>Technology</th>
    <th>Purpose</th>
  </tr>
  <tr>
    <td>Docker</td>
    <td>Containerization of application</td>
  </tr>
  <tr>
    <td>Kubernetes</td>
    <td>Container orchestration</td>
  </tr>
  <tr>
    <td>ArgoCD</td>
    <td>GitOps continuous deployment tool</td>
  </tr>
  <tr>
    <td>GitHub</td>
    <td>Source code and configuration repository</td>
  </tr>
  <tr>
    <td>Minikube / AWS EKS</td>
    <td>Kubernetes cluster environment</td>
  </tr>
  <tr>
    <td>Linux</td>
    <td>DevOps environment</td>
  </tr>
</table>

---
### Launch EC2 Instance

![](./img/Screenshot%20(354).png)

### Project Structure

![](./img/Screenshot%20(355).png)

### Step 1: Containerize the Application
Create a Dockerfile to package the application into a container image.
```
FROM python:3.9
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python","app.py"]
```

Build Docker image:
```
docker build -t <dockerhub-username>/gitops-app:v1 .
```
Push image to DockerHub:
```
docker push <dockerhub-username>/gitops-app:v1
```
---
### Step 2: Deploy Application on Kubernetes

Deployment Configuration

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitops-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gitops-app
  template:
    metadata:
      labels:
        app: gitops-app
    spec:
      containers:
      - name: gitops-app
        image: chouguleaniket/gitops-app:v1
        ports:
        - containerPort: 5000
```

Service Configuration

```
apiVersion: v1
kind: Service
metadata:
  name: gitops-service
spec:
  selector:
    app: gitops-app
  ports:
    - port: 80
      targetPort: 5000
  type: NodePort
  ```

Apply manifests:
```
kubectl apply -f kubernetes/
```
![](./img/Screenshot%20(353).png)

---
### Step 3: Install ArgoCD
Create namespace:
```
kubectl create namespace argocd
```
Install ArgoCD:
```
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify installation:
```
kubectl get pods -n argocd
```
Expose ArgoCD UI:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Access ArgoCD Dashboard:
```
https://localhost:8080
```
Default credentials:
```
Username: admin
```
Retrieve password:
```
kubectl get secret argocd-initial-admin-secret \
-n argocd -o jsonpath="{.data.password}" | base64 -d
```

![](./img/Screenshot%20(349).png)

---
### Step 4: Configure ArgoCD Application
Create an ArgoCD Application resource to connect the Git repository.

Apply configuration:
```
kubectl apply -f argocd/application.yaml
```
![](./img/Screenshot%20(350).png)

---
### Step 5: GitOps Workflow
The deployment process follows GitOps principles:

1) Developer updates application code
2) Docker image is rebuilt and pushed to the container registry
3) Kubernetes manifests are updated with the new image version
4) Changes are committed and pushed to GitHub
5) ArgoCD detects repository changes
6) ArgoCD automatically synchronizes the Kubernetes cluster
7) Kubernetes updates running pods with the new version

This workflow ensures fully automated deployments without manual intervention.

---
### Testing Automatic Deployment
Update application code.

Rebuild and push new Docker image:
```
docker build -t <dockerhub-username>/gitops-app:v2 .  
docker push <dockerhub-username>/gitops-app:v2
```
Update image tag in deployment.yaml:
```
image: <dockerhub-username>/gitops-app:v2
```
Commit changes:
```
git add . 
git commit -m "Update application version" 
git push
```
![](./img/Screenshot%20(351).png)

ArgoCD will automatically detect the change and deploy the updated version.

![](./img/Screenshot%20(352).png)

---
### Optional Enhancements
This project can be extended with:
* Helm-based deployments
* Kubernetes Ingress Controller
* Horizontal Pod Autoscaler (HPA)
* CI pipeline using Jenkins or GitHub Actions
* Monitoring using Prometheus and Grafana

---
### Benefits of GitOps
* Declarative infrastructure management
* Automated deployments
* Version-controlled infrastructure
* Simplified rollback using Git history
* Improved system reliability and auditability
---
### Conclusion
This project successfully demonstrates the implementation of GitOps principles for Kubernetes deployments using ArgoCD. By integrating Git, Docker, Kubernetes, and ArgoCD, the system enables automated, reliable, and scalable application deployments.# gitops-project
