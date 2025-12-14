# k8s-devops-demo
This project demonstrates a production-style DevOps workflow using Kubernetes, CI/CD concepts, and monitoring — built with low cost and simplicity in mind.

---

##  Project Overview

The goal of this project is to:

* Deploy a containerized application on Kubernetes
* Expose it using an Ingress controller
* Validate self-healing and rolling updates
* Monitor infrastructure and workloads using Prometheus and Grafana


---

##  Architecture

```
Host (EC2 / WSL / VM)
└── k3s (Single-node Kubernetes)
    ├── Application Pods (Flask App)
    ├── Service (ClusterIP)
    ├── Traefik Ingress (k3s default)
    ├── Prometheus (metrics collection)
    ├── Grafana (metrics visualization)
    └── kube-state-metrics (cluster metrics)
```

---

##  Technology Stack

* **Kubernetes:** k3s
* **Container Runtime:** containerd (bundled with k3s)
* **Application:** Python Flask
* **Containerization:** Docker
* **Ingress Controller:** Traefik (default with k3s)
* **Monitoring:** Prometheus, Grafana
* **Package Manager:** Helm

---

##  Repository Structure

```
.
├── app/
│   ├── app.py
│   └── Dockerfile
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── monitoring/
│   └── values.yaml
└── README.md
```

---

## ⚙️ Prerequisites

* Linux VM / EC2 / WSL2
* kubectl
* Docker
* Helm

---

##  Step 1: Install k3s

```bash
curl -sfL https://get.k3s.io | sh -
```

Configure kubectl access:

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

Verify:

```bash
kubectl get nodes
```

---

##  Step 2: Build and Push Application Image

```bash
cd app
docker build -t <dockerhub-username>/devops-demo:1.0 .
docker login
docker push <dockerhub-username>/devops-demo:1.0
```

---

##  Step 3: Deploy Application to Kubernetes

Create namespace:

```bash
kubectl create namespace devops-demo
```

Apply manifests:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
```

Verify:

```bash
kubectl get pods -n devops-demo
kubectl get ingress -n devops-demo
```

Access the app using the **Ingress IP**:

```bash
http://<INGRESS_IP>
```

---

##  Step 4: Validate Self-Healing

Delete a pod:

```bash
kubectl delete pod -n devops-demo -l app=devops-demo
```

Observe:

* Pods are automatically recreated
* Application remains accessible

This demonstrates **Kubernetes self-healing**.

---

##  Step 5: Rolling Update Test

Build and push a new image tag:

```bash
docker build -t <dockerhub-username>/devops-demo:2.0 .
docker push <dockerhub-username>/devops-demo:2.0
```

Update deployment:

```bash
kubectl set image deployment/devops-demo \
  app=<dockerhub-username>/devops-demo:2.0 \
  -n devops-demo
```

Watch rollout:

```bash
kubectl rollout status deployment/devops-demo -n devops-demo
```

---

##  Step 6: Install Prometheus & Grafana

Add Helm repo:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Create monitoring namespace:

```bash
kubectl create namespace monitoring
```

Install stack:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f monitoring/values.yaml
```

Verify:

```bash
kubectl get pods -n monitoring
```

---

##  Step 7: Access Grafana

Port-forward Grafana:

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Open browser:

```
http://localhost:3000
```

Credentials:

* Username: `admin`
* Password: `admin`

---

##  Recommended Dashboards

Import these dashboards in Grafana:

* **1860** – Node Exporter Full
* **8588** – Kubernetes Deployment Overview

---

##  Step 8: Observe Failures in Grafana

Delete pods again:

```bash
kubectl delete pod -n devops-demo -l app=devops-demo
```

Observe in Grafana:

* Pod restart count
* Replica recovery
* CPU and memory spikes

This validates **monitoring + self-healing** together.

---

##  What This Project Demonstrates

* Kubernetes workload deployment
* Ingress-based traffic routing
* Rolling updates and self-healing
* Cluster and node observability
* Production-style monitoring

---

##  Notes

* This is a **single-node cluster** for demo purposes
* In production, monitoring typically runs on dedicated nodes or clusters
* K3s used because it is lightweight. Another option is minikube.
---

## ✅ Outcome

This project demonstrates the ability to **deploy, operate, monitor, and troubleshoot Kubernetes workloads**, aligning closely with real-world DevOps and SRE responsibilities.

---

## 📄 License

MIT
