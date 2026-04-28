# Home Lab SRE Platform & Sunday Pickems Application

## Overview

This project represents a production-style Site Reliability Engineering (SRE) environment built in a home lab, supporting a multi-tenant Sunday Pickems application.

It demonstrates real-world infrastructure, observability, and automation practices including Kubernetes, GitOps, monitoring, logging, and CI/CD pipelines.

---

## 🧱 Architecture Overview

- **Platform**: Proxmox (virtualization)
- **Kubernetes**: k3s (HA control plane)
- **Ingress**: Traefik (lab.local domain)
- **Storage**: Longhorn (persistent volumes)
- **CI/CD**: GitHub Actions → Harbor → Kubernetes
- **DNS**: Pi-hole (internal resolution)

### Security & Secrets
- **Vault** → Injects secrets into Kubernetes workloads (environment variables)
- Secrets not stored in code or manifests

### TLS & Certificates
- **cert-manager** → Automates certificate management
- Self-signed ClusterIssuer for internal lab traffic

---

## ⚙️ Core Components

## 🔒 Security & Secret Management

Secrets are managed using HashiCorp Vault and injected into Kubernetes workloads at runtime.

### Implementation
- Vault stores application secrets (DB credentials, API keys)
- Kubernetes deployments reference secrets via environment variables
- Secrets are not hardcoded or stored in Git

### Benefits
- Centralized secret management
- Reduced risk of credential exposure
- Closer alignment with production security practices

### 🏈 NFL Pick’em Application
- Flask-based web application
- Multi-tenant (multiple groups/pools)
- Confidence-based scoring system
- Admin dashboards and group management
- Background scheduler (APScheduler) for automation
- PostgreSQL backend (local + Render in production)

---

### ☸️ Kubernetes Platform
- 3 control plane nodes
- 3 worker nodes
- Load-balanced API server
- GitOps-style deployments
- Internal-only cluster networking

---

### 📊 Observability Stack
- **Prometheus** → Metrics collection (/metrics endpoint)
- **Grafana** → Dashboards and visualization
- **Loki + Promtail** → Log aggregation
- **Alertmanager** → Email alerting
- **TLS via cert-manager** → Secure endpoints 
---

### 🔁 CI/CD Pipeline
- GitHub Actions builds Docker images
- Images pushed to private Harbor registry
- Kubernetes deployments updated with new image tags

---

## 📡 Observability & Monitoring

### Metrics
- Application metrics exposed via `/metrics`
- Tracks request rates, errors, and system behavior
- Custom metric:
  - `football_current_nfl_week`

### Logging
- Centralized logging via Loki
- Promtail collects logs from Kubernetes workloads

### Alerting
- Email-based alerts via Alertmanager
- Thresholds currently being tuned

---
## 🧠 SRE Practices Demonstrated

- Infrastructure as Code (Kubernetes manifests / GitOps)
- Monitoring and alerting design
- Distributed system troubleshooting
- Service decomposition (web + scheduler)
- Observability integration (metrics + logs)
- CI/CD automation pipeline
- Multi-tenant application architecture

---

## ⚠️ Known Gaps / Improvements

- No formal SLIs/SLOs defined yet
- Backup strategy not implemented
- External uptime monitoring not configured
- Alert thresholds need refinement

---

## 🗺️ Roadmap

### Short Term
- Define SLIs/SLOs for application reliability
- Implement SMS notifications (Twilio)
- Improve scheduler observability

### Medium Term
- Add backup and recovery strategy
- External monitoring of production app
- Harden security (Vault integration)

### Long Term
- Public deployment improvements
- Scale testing and performance tuning

---

## 🔗 Related Repositories

- Kubernetes GitOps Repo:
  https://github.com/stevenbarnes2017/k3s-proxmox-gitops

---

## 🎯 Purpose

This project is designed to simulate real-world SRE responsibilities, including:

- Operating distributed systems
- Managing observability platforms
- Automating deployments
- Improving system reliability

---

## 👤 Author

Steven Barnes  
Senior Infrastructure / SRE Engineer
