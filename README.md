# 1 Project Title: Deploy a Containerized Web Application with CI/CD and Monitoring

## Objective

Set up and deploy a simple web application (WordPress) using Docker and Kubernetes, automate its deployment through a CI/CD pipeline, and integrate basic monitoring using Prometheus and Grafana.

## Tools and Technology

- Containerization: Docker
- Orchestration: Kubernetes (Minikube, K3s, or cloud-based)
- CI/CD: Jenkins, GitHub Actions, or GitLab CI
- IaC: Terraform or Ansible
- Monitoring: Prometheus and Grafana
- Cloud (free tier): AWS

## Project Deliveries

Part 1: Application Setup (WordPress)

1. Web Application: you can use a public Helm chart or any other method that is publicly available.
2. The application should be accessible from the browser.

Part 2: Kubernetes Deployment

1. Deploy the Application changes via CI & CD pipeline: Use Jenkins or any other CI&CD to deploy the application changes.
2. Changes should be reflected in the browser.

Part 3: Monitoring and Observability

1. Prometheus and Grafana:
   a. Deploy Prometheus and Grafana on the Kubernetes cluster.
   b. Create a basic Prometheus configuration to scrape metrics from the Kubernetes nodes and pods.
   c. Set up a simple Grafana dashboard to display metrics (e.g., CPU/Memory usage).
   d. Set up a simple Grafana dashboard to display application metrics.

## 3. System Requirements

- 8 GB RAM minimum
- 4 CPU cores
- Stable internet connection
- Docker pre-installed
- Git installed

## 4. Environment Setup

4.1 Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

Logout and login again.

4.2 Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Start Minikube:

```bash
minikube start --driver=docker --cpus=4 --memory=4096
```

4.3 Install kubectl

```bash
sudo apt install -y kubectl
```

4.4 Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## 5. Kubernetes Deployment

Your repository contains these files:

```
deployment/
 ├── namespace.yaml
 ├── mysql-deployment.yaml
 └── wordpress-deployment.yaml
```

5.1 Apply Kubernetes Namespace

```bash
kubectl apply -f deployment/namespace.yaml
```

5.2 Deploy MySQL

```bash
kubectl apply -f deployment/mysql-deployment.yaml -n wordpress-cicd
```

Check MySQL pod:

```bash
kubectl get pods -n wordpress-cicd
```

5.3 Deploy WordPress

```bash
kubectl apply -f deployment/wordpress-deployment.yaml -n wordpress-cicd
```

Check services:

```bash
kubectl get svc -n wordpress-cicd
```

Get Minikube IP:

```bash
minikube ip
```

Access WordPress:

```
http://<minikube-ip>:<nodeport>
```

## 6. CI/CD Setup (Jenkins)

6.1 Run Jenkins in Docker

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

6.2 Install required Jenkins plugins

- Git
- GitHub
- Pipeline
- GitHub Integration
- GitHub API
- GitHub Branch Source

6.3 Add GitHub Credentials

Navigate:

Jenkins → Manage Jenkins → Credentials → Global → Add Credentials

Type: Username + Password

Username: your GitHub username

Password: GitHub Personal Access Token

ID example: `github-creds`

6.4 Configure Jenkins Job

Inside your Pipeline job:

Pipeline → SCM → Git

Repository URL:

```
https://github.com/RjyavardhanSingh/wordpress-k8s-cicd.git
```

Credentials: `github-creds`

Branch: `*/main`

6.5 Enable GitHub Webhooks

GitHub → Repository → Settings → Webhooks → Add Webhook

Payload URL:

```
https://<ngrok-url>/github-webhook/
```

Content type: `application/json`

Events: Just the push event

6.6 Start ngrok to expose Jenkins

```bash
ngrok http 8080
```

Copy the URL and update the webhook.

## 7. Jenkinsfile Pipeline Explanation

Steps performed:

1. Checkout repository
2. Apply namespace, MySQL, WordPress YAML
3. Verify pods
4. Finish pipeline

This ensures that any push to GitHub automatically redeploys WordPress.

## 8. Monitoring Setup

8.1 Install Prometheus + Grafana

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace
```

8.2 Access Prometheus

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090
```

Open:

```
http://localhost:9090
```

8.3 Access Grafana

```bash
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

Open:

```
http://localhost:3000
```

Default credentials:

- username: `admin`
- password: `prom-operator`

## 9. Prometheus Queries Used

Pod information:

```
kube_pod_info{namespace="wordpress-cicd"}
```

CPU requests:

```
kube_pod_container_resource_requests_cpu_cores{namespace="wordpress-cicd"}
```

CPU usage (live):

```
rate(container_cpu_usage_seconds_total{namespace="wordpress-cicd", container!="POD"}[5m])
```

Memory usage:

```
container_memory_working_set_bytes{namespace="wordpress-cicd", container!="POD"}
```

Pod restarts:

```
kube_pod_container_status_restarts_total{namespace="wordpress-cicd"}
```

## 10. Grafana Dashboards

Dashboards included:

- CPU usage per pod
- Memory usage per pod
- Pod restarts
- Pod status
- WordPress deployment resource usage

These are built from Prometheus queries.

## 11. Final Notes

- CI/CD pipeline is fully automated through GitHub webhook
- WordPress redeploys after each git push
- Monitoring stack is fully functional
- All YAML files are stored in the `deployment/` folder
- Project runs 100% locally on Minikube

