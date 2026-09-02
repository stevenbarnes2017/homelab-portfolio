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
- Private node and service networks with selected applications published through Cloudflare Tunnel
- Lightweight Kubernetes (k3s)

---

## 🌐 Networking

### Ingress Controller

- **Traefik**
  - Handles HTTP/HTTPS routing
  - Ingress class: `traefik`

Traefik is the only discovered Kubernetes `LoadBalancer` service and is available on the LAN at `10.0.0.119` over ports 80 and 443. Public application traffic does not depend on direct Xfinity TCP 80/443 forwarding.

### Public Application Path

Selected applications use this verified path:

```text
Internet -> Cloudflare -> Cloudflare Tunnel -> ClusterIP service -> application pod
```

`cloudflared` runs as a two-replica Deployment in the `immich` namespace. Public route details are maintained in `docs/security/cloudflare-ingress.md`.

### Pod Networking and Segmentation

The verified Kubernetes network path is:

```text
Flannel data plane -> kube-router NetworkPolicy enforcement -> workload
```

NetworkPolicy enforcement is active. Current coverage includes ArgoCD, Immich Redis, and the Celestial pilot, but most namespaces have no policy and therefore remain default-allow for east-west traffic. Public-facing application workloads and sensitive management workloads still share the cluster without broad namespace-level isolation.

Celestial is the first verified namespace-level segmentation pilot. It uses default-deny ingress and egress with explicit allowances for Cloudflare Tunnel ingress, CoreDNS, PostgreSQL at `10.0.0.129:5432`, and external HTTPS APIs. Generic Internet HTTPS access excludes RFC1918 destinations unless explicitly allowed. An unrelated pod was unable to connect directly, while this production path remained healthy:

```text
Vercel frontend -> celestial-api.sundaypickems.com -> Cloudflare -> Cloudflare Tunnel -> Celestial Service -> Celestial API pod
```

The remaining Traefik allowance for the unused `celestial.lab.local` ingress is a cleanup candidate rather than a required production path. Broader segmentation must account for cross-boundary dependencies before enforcement:

- Open WebUI (`ai`) to Ollama at `10.0.0.41:11434`
- Workloads using PostgreSQL at `10.0.0.129`
- `cloudflared` (`immich`) to published services across namespaces
- Prometheus cross-namespace scraping
- DNS to CoreDNS in `kube-system`
- Media/NFS traffic, Harbor, Longhorn, and other cross-namespace services

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

### Hermes / Local LLM

- Open WebUI runs in Kubernetes as the Hermes user interface.
- `hermes.barnesfamily-pics.online` reaches Open WebUI through Cloudflare Tunnel and `open-webui.ai.svc.cluster.local:8080`.
- Ollama runs separately on a Windows home PC at the observed LAN address `10.0.0.41:11434` to use an NVIDIA RTX 2070 SUPER.
- Open WebUI communicates with Ollama across the LAN.

---

## ⚠️ Known Gaps / Improvements

- No automated backup strategy (Longhorn volumes)
- No external monitoring of cluster
- NetworkPolicy enforcement is active, but most namespaces remain default-allow
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

The cluster nodes and service networks remain private, while only explicitly routed applications are published through Cloudflare Tunnel. This distinction preserves a private infrastructure plane without describing the application plane as entirely unexposed.

Administrative applications including ArgoCD, Longhorn, Vault, Harbor, Prometheus, Grafana, and Alertmanager should be classified as management-plane services during future network segmentation.

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
