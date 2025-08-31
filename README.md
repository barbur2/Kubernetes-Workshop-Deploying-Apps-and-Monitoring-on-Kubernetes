# Kubernetes WordPress Deployment with Monitoring
## 📌 Overview
This project migrates the WordPress + MySQL stack from Docker Compose into Kubernetes.
It also adds an NGINX Ingress Controller for routing and a Grafana/Prometheus stack for monitoring.
Definition of Done:
WordPress application is deployed and reachable.
MySQL database runs with persistent storage.
Grafana dashboard shows container uptime metrics.
All Kubernetes manifests are included in this repo (mysql.yaml, wordpress.yaml).
Clear deployment steps provided.

## 🚀 Deployment Steps
1. Prerequisites
EC2 instance with Minikube installed.
AWS ECR with mysql:latest and wordpress:latest images pushed.
kubectl + helm installed on EC2.

2. Install NGINX Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
kubectl get pods -n ingress-nginx

3. Deploy MySQL
Apply the mysql.yaml file:
kubectl apply -f mysql.yaml
Verify:
kubectl get statefulsets
kubectl get pvc
kubectl get pods -l app=mysql

4. Deploy WordPress
Apply the wordpress.yaml file:
kubectl apply -f wordpress.yaml
Verify:
kubectl get deployments
kubectl get pods -l app=wordpress
kubectl get ingress

5. Access WordPress
Update /etc/hosts:
<minikube_ip> wordpress.local
Find IP:
minikube ip
Test:
curl http://wordpress.local
or open in browser → WordPress installation screen should appear.

6. Install Prometheus + Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace
Verify:
kubectl get pods -n monitoring

7. Access Grafana
Port-forward Grafana service:
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80
From your local machine, open SSH tunnel:
ssh -i /path/to/key.pem -L 3000:localhost:3000 ec2-user@<EC2-PUBLIC-IP>
Open in browser:
http://localhost:3000
Get admin password:
kubectl -n monitoring get secret kube-prometheus-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 -d ; echo
Default username: admin

8. Create Grafana Dashboard
Go to Dashboards → New → New Panel.
Select Prometheus data source.
Use query:
kube_pod_container_status_running
This shows if containers are up (1) or down (0).
Save dashboard as WordPress Monitoring.

## 📂 Repository Structure
.
├── mysql.yaml         # MySQL StatefulSet, PVC, Service
├── wordpress.yaml     # WordPress Deployment, Service, Ingress
└── README.md          # Deployment instructions

## ✅ Result
WordPress reachable at http://wordpress.local
Persistent MySQL database
Grafana dashboard with uptime panel
