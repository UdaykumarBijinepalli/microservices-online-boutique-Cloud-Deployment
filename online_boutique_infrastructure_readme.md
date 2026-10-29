# 🛒 Enterprise 11-Tier Polyglot Microservices Cloud Infrastructure

[![Terraform Validate](https://img.shields.io/badge/Terraform-1.8.0-623CE4?logo=terraform)](https://www.terraform.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.30-326CE5?logo=kubernetes)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-v3.14-0F1689?logo=helm)](https://helm.sh/)
[![Observability](https://img.shields.io/badge/Monitoring-Prometheus%20%26%20Grafana-F46800?logo=grafana)](https://grafana.com/)
[![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=githubactions)](https://github.com/features/actions)

A complete, production-grade cloud-native deployment of Google Cloud's reference e-commerce platform (**Online Boutique**). This project demonstrates infrastructure automation, modular packaging, high-availability routing, and full-stack observability across 11 polyglot microservices.

---

## 🏗 System Architecture

The architecture consists of 11 loosely coupled microservices communicating internally via high-performance **gRPC** and caching session state in **Redis**:

| Microservice | Runtime / Language | Port | Primary Responsibility |
| :--- | :--- | :--- | :--- |
| **frontend** | Go | `8080` | Serves web UI and acts as the internal API Gateway routing traffic to backend services. |
| **cartservice** | C# (.NET 8) | `7070` | Manages shopping cart items, quantities, and persistent state backed by Redis. |
| **productcatalogservice** | Go | `3550` | Read-only service providing catalog lookup, product details, and category queries. |
| **currencyservice** | Node.js | `7000` | Fetches real-time exchange rates and converts product pricing across global currencies. |
| **paymentservice** | Node.js | `50051` | Simulates mock credit card processing and payment authorization logic. |
| **shippingservice** | Go | `50051` | Computes real-time shipping costs and generates tracking numbers. |
| **emailservice** | Python | `8080` | Dispatches simulated HTML order confirmation emails upon transaction checkout. |
| **checkoutservice** | Go | `5050` | Distributed transaction coordinator orchestrating payment, inventory, and order dispatch. |
| **recommendationservice** | Python | `8080` | Computes dynamic product recommendations based on items viewed or added to cart. |
| **adservice** | Java | `9555` | Generates contextual promotional banners matching item category tags. |
| **loadgenerator** | Python (Locust) | N/A | Simulates concurrent customer browsing & orders |

---

## ⚙️ DevOps & Cloud Toolchain

- **Infrastructure as Code (IaC):** Modular Terraform configurations provisioning a multi-AZ AWS VPC (public subnets, route tables, Internet Gateway) and a managed Amazon EKS cluster with 2 worker nodes (`t3.medium`).
- **Package Management (Helm):** Modular Helm chart replacing 1,000+ lines of raw static YAML, parameterizing resource allocations (`requests`/`limits`), image tags, and service manifests via `values.yaml`.
- **Traffic Routing (Ingress):** Nginx Ingress Controller managing incoming external HTTP traffic and routing path prefixes directly to the frontend service.
- **Full-Stack Observability:** Deployed `kube-prometheus-stack` (Prometheus Operator, Grafana, Node-Exporter, and Kube-State-Metrics) collecting live container metrics, CPU quotas, memory footprints, and network telemetry under Locust load.
- **CI/CD Pipeline:** GitHub Actions workflow executing automated linting (`terraform fmt`, `terraform validate`, `helm lint`) and continuous cluster deployments.

---

## 📊 Live Observability & Performance Metrics

Telemetry captured in Grafana (`Kubernetes / Compute Resources / Namespace (Pods)`) illustrating real-time CPU and memory utilization curves driven by continuous simulated transactions from `loadgenerator`:

![Observability Telemetry](docs/images/grafana-metrics.png)

---

## 📁 Repository Structure

```text
.
├── .github/
│   └── workflows/
│       └── deploy.yml           # GitHub Actions CI/CD Pipeline
├── docs/
│   └── images/
│       └── grafana-metrics.png  # Cluster telemetry screenshots
├── helm-chart/
│   ├── templates/               # Kubernetes deployment & service templates
│   ├── Chart.yaml               # Helm metadata & release versioning
│   └── values.yaml              # Parameterized CPU/Memory requests & images
├── terraform/
│   ├── eks.tf                   # EKS Cluster, IAM roles, and managed node groups
│   ├── main.tf                  # Terraform AWS provider setup
│   ├── variables.tf             # Parameterized region, cluster name & instance sizes
│   └── vpc.tf                   # Multi-AZ VPC, subnets, route tables, and gateways
├── .gitignore                   # Ignore terraform binaries, plugins & state files
└── README.md                    # System architecture documentation
```

---

## 🚀 Quickstart & Deployment

### 1. Provision AWS Infrastructure via Terraform
```bash
cd terraform
terraform init
terraform plan
terraform apply -auto-approve
```

### 2. Connect Local Kubectl to Amazon EKS
```bash
aws eks update-kubeconfig --region ap-south-1 --name boutique-eks-cluster
```

### 3. Deploy Stack via Helm
```bash
helm upgrade --install online-boutique ./helm-chart \
  --namespace default \
  --create-namespace
```

### 4. Deploy Ingress Controller & Observability
```bash
# Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx

# Observability (Prometheus & Grafana)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack
```

---

## 🔐 Engineering Highlights & Problem-Solving

* **Scheduler Optimization:** Resolved `0/1 nodes available: Insufficient cpu` scheduler bottlenecks on resource-constrained compute by profiling baseline loads and tuning `resources.requests.cpu` across all 11 polyglot runtimes.
* **Service Decoupling:** Replaced hardcoded service endpoints with Kubernetes DNS resolution over gRPC service contracts.
* **Clean State Isolation:** Decoupled transient shopping cart storage into dedicated Redis instances with zero state retention in microservice runtimes.