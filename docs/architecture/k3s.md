# Kubernetes Architecture (k3s Home Lab)

## Overview

This Kubernetes platform is a self-hosted k3s cluster running on Proxmox, designed to simulate a production-grade environment for deploying and operating distributed applications.

The cluster supports the Sunday Pickem's application along with persistent storage, a full observability stack, CI/CD pipeline, and secure secret management.

---

## 🧱 Cluster Topology

### Nodes

- **Control Plane Nodes (3)**
  - control-plane-01
  - control-plane-02
  - control-plane-03
  - 
- **Worker Nodes (3)**
  - worker-01
  - worker-02
  - worker-03

- **Load Balancer**
  - k3s-lb-01 (API endpoint)

---

### Key Characteristics

- High availability control plane
- Internal-only cluster (no external exposure)
- Lightweight Kubernetes (k3s)

---

## 🌐 Networking

### Ingress Controller

- **Traefik**
  - Handles HTTP/HTTPS routing
  - Ingress class: `traefik`

---

### DNS

- Managed via Pi-hole
- Internal domain:
  - `*.lab.local`

### Example Services

- `argocd.lab.local`
- `grafana.lab.local`
- `prometheus.lab.local`
- `football.lab.local`
- `longhorn.lab.local`
- `harbor.lab.local`

---

## 💾 Storage

### Longhorn

- Distributed block storage
- Persistent volumes for:
  - Prometheus
  - Grafana
  - Alertmanager

### Benefits

- Data persistence across pod restarts
- Simplified storage management

---

## 🔐 Security & Secrets

### Vault Integration

- HashiCorp Vault used for secret management
- Secrets injected into pods via environment variables

#### Use Cases

- Database credentials
- API keys
- Email service credentials

---

## 🔐 TLS & Certificate Management

### cert-manager

- Automates certificate lifecycle
- Uses self-signed ClusterIssuer for internal services

### Usage

- Secures ingress endpoints
- Enables HTTPS for internal services

---

## 📊 Observability Stack

### Components

- **Prometheus**
  - Metrics collection
  - Scrapes `/metrics` endpoints

- **Grafana**
  - Dashboards and visualization

- **Loki**
  - Log aggregation

- **Promtail**
  - Log collection agent

- **Alertmanager**
  - Sends alerts via email

---

### Data Flow

- Applications expose `/metrics`
- Prometheus scrapes metrics
- Logs collected by Promtail → Loki
- Grafana visualizes both metrics and logs
- Alertmanager triggers notifications

---

## 🔁 CI/CD Pipeline

### Flow

1. Developer pushes code to GitHub
2. GitHub Actions builds Docker image
3. Image pushed to Harbor (private registry)
4. Kubernetes deployment updated with new image
5. Pods restart with updated version

---

### Image Management

- Harbor hosted internally
- Authenticated access
- Versioned and `latest` tagging strategy

---

## 🚀 Workloads

### Sunday Pickem's Application

- `football-web` (Flask + Gunicorn)
- `football-scheduler` (APScheduler worker)

---

### Observability

- kube-prometheus-stack
- loki-stack

---

### Supporting Services

- ArgoCD (GitOps)
- Prometheus exporters (node, kube-state, etc.)

---

## ⚠️ Known Gaps / Improvements

- No automated backup strategy (Longhorn volumes)
- No external monitoring of cluster
- Limited network segmentation
- Vault usage could be expanded (not fully standardized)

---

## 🧠 Design Decisions

### Why k3s?

- Lightweight Kubernetes for home lab
- Easier management than full kubeadm

---

### Why Separate Scheduler?

- Prevents blocking web requests
- Improves reliability
- Aligns with microservice principles

---

### Why Internal-Only Cluster?

- Reduces attack surface
- Simplifies networking and security

---

## 🔭 Future Improvements

### Short Term

- Implement backup strategy (Longhorn snapshots or external)
- Define SLIs/SLOs
- Improve alert tuning

---

### Medium Term

- External uptime monitoring
- Expand Vault integration
- Add network policies

---

### Long Term

- Multi-cluster or cloud integration
- Load testing and scaling
- Zero-downtime deployments

---

## 📍 Summary

This Kubernetes platform serves as a fully functional SRE environment, supporting application workloads, observability, automation, and security practices aligned with real-world infrastructure operations.
