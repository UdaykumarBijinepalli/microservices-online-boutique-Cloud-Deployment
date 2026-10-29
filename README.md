# Enterprise 11-Tier Polyglot Microservices Cloud Infrastructure

An end-to-end cloud-native deployment of Google Cloud's reference e-commerce platform (**Online Boutique**), orchestrated across 11 polyglot microservices, containerized with Docker, packaged using modular Helm charts, provisioned via Terraform IaC, and monitored using Prometheus & Grafana.

---

## 🏗 System Architecture

The platform consists of 11 loosely coupled microservices communicating internally over high-performance **gRPC** and caching session data in **Redis**:

| Service | Runtime / Framework | Port | Purpose |
| :--- | :--- | :--- | :--- |
| **frontend** | Go | `8080` | Web UI & API Gateway routing to backend services |
| **cartservice** | C# (.NET 8) | `7070` | Shopping cart state management backed by Redis |
| **productcatalogservice** | Go | `3550` | Read-only product listing and search |
| **currencyservice** | Node.js | `7000` | Real-time exchange rate conversions |
| **paymentservice** | Node.js | `50051` | Mock credit card transaction handling |
| **shippingservice** | Go | `50051` | Real-time shipping cost quotes and tracking IDs |
| **emailservice** | Python | `8080` | Mock order confirmation notification dispatcher |
| **checkoutservice** | Go | `5050` | Distributed transaction orchestrator for checkouts |
| **recommendationservice** | Python | `8080` | Product recommendation engine |
| **adservice** | Java | `9555` | Contextual ad banner generator |
| **loadgenerator** | Python (Locust) | N/A | Simulates concurrent customer browsing & orders |

---

## ⚙️ DevOps & Cloud Toolchain

1. **Infrastructure as Code (IaC):** Modular Terraform configs provisioning a multi-AZ AWS VPC and Amazon EKS cluster with 2 worker nodes (`t3.medium`).
2. **Kubernetes Packaging (Helm):** Parameterized Helm chart replacing 1,000+ lines of monolithic YAML, centralizing resource allocations, ConfigMaps, and service definitions.
3. **Traffic Ingress:** Ingress Nginx Controller mapping external HTTP traffic to the `frontend` service.
4. **Full-Stack Observability:** Deployed `kube-prometheus-stack` (Prometheus & Grafana) collecting real-time CPU, memory, and gRPC network latency under synthetic Locust traffic.
5. **CI/CD Pipeline:** GitHub Actions workflow executing automated linting (`terraform fmt`, `terraform validate`, `helm lint`).

---

## 🚀 Quickstart

### 1. Provision Infrastructure
```bash
cd terraform
terraform init
terraform plan
terraform apply -auto-approve
aws eks update-kubeconfig --region ap-south-1 --name boutique-eks-cluster
helm upgrade --install online-boutique ./helm-chart \
  --namespace default \
  --create-namespace
```
