# 🏠 Homelab & Platform Portfolio

> **Steven Barnes** · Infrastructure Engineer · Platform Engineer · SRE
>
> [LinkedIn](https://www.linkedin.com/in/steven-barnes-5b986856/) · [GitHub](https://github.com/stevenbarnes2017) · [Sunday Pickems](https://sundaypickems.com)

---

## Overview

This repository documents my homelab infrastructure, DevOps practices, and the Sunday Pickems platform — a real-world, multi-tenant SaaS application I designed, built, and operate end-to-end.

The lab serves as a personal learning environment built to enterprise standards — everything here reflects skills and practices directly applicable to production infrastructure engineering roles.

---

## 🚀 Projects

### Sunday Pickems Platform

A production Flask web application for running NFL confidence-based pick'em pools. Multi-tenant architecture supporting multiple independent groups with role-based access control.

- **Live site:** [sundaypickems.com](https://sundaypickems.com)
- **Stack:** Python/Flask, PostgreSQL, Docker, Kubernetes, Vault, Prometheus
- **Users:** Real users, live NFL season data, automated email via Brevo
- [Full documentation →](docs/service/README.md)

### Homelab Kubernetes Cluster

A production-grade Kubernetes cluster running on Proxmox — fully automated from bare metal to running applications using Terraform and Ansible.

- **Cluster:** 1 load balancer, 3 control planes, 3 workers
- **GitOps:** ArgoCD synced to GitHub as source of truth
- **Storage:** Longhorn persistent storage
- **Observability:** Prometheus, Grafana, Loki, Alertmanager
- **Security:** HashiCorp Vault for secret management
- **Registry:** Harbor private container registry
- [Full documentation →](docs/architecture/k3s.md)

---

## 🏗️ Infrastructure Stack

### Compute & Virtualization

| Technology | Role |
|---|---|
| Proxmox VE | Hypervisor — 2 node cluster (steven, lab2) |
| Ubuntu 22.04 | VM OS via cloud-init templates |
| Debian 12 | LXC container OS |
| PostgreSQL VM | Dedicated Ubuntu VM running PostgreSQL for the Pickems app |

### Kubernetes & GitOps

| Technology | Role |
|---|---|
| Kubernetes | Container orchestration — 7 node cluster |
| ArgoCD | GitOps — GitHub as source of truth |
| Longhorn | Persistent storage |
| Harbor | Private container registry |
| HashiCorp Vault | Secret management |

### Observability

| Technology | Role |
|---|---|
| Prometheus | Metrics collection |
| Grafana | Dashboards and visualization |
| Loki | Log aggregation |
| Alertmanager | Alert routing |
| Uptime Kuma | Service uptime monitoring |

### Infrastructure as Code

| Technology | Role |
|---|---|
| Terraform | VM and LXC provisioning on Proxmox |
| Ansible | Configuration management, patching |
| GitHub Actions | CI/CD pipelines |

### Networking & Security

| Technology | Role |
|---|---|
| WireGuard | VPN — secure remote access to homelab |
| Pi-hole | Local DNS resolver |
| Homarr | Homelab service dashboard |

### Application Stack

| Technology | Role |
|---|---|
| Python / Flask | Web application framework |
| PostgreSQL | Primary database — dedicated VM |
| Docker | Containerization |
| Render | Production hosting |
| Brevo | Transactional email |

---

## 📁 Repository Structure

```
homelab-portfolio/
├── docs/
│   ├── architecture/          # Infrastructure architecture docs
│   │   ├── README.md          # Architecture overview
│   │   └── k3s.md             # Kubernetes cluster deep dive
│   ├── diagrams/              # Architecture diagrams
│   ├── roadmap/               # Project roadmap and planning
│   │   └── README.md          # Current roadmap
│   ├── runbooks/              # Operational runbooks
│   │   ├── bootstrap-k3s-cluster.md
│   │   └── scheduler-not-running.md
│   └── service/               # Application documentation
│       └── README.md          # Sunday Pickems platform docs
└── README.md                  # This file
```

---

## 🗺️ Architecture Overview

```
Internet
    │
    ▼
Xfinity Gateway (NAT · Port 51820 UDP forwarded)
    │
    ▼
Homelab Network (10.0.0.0/24)
    │
    ├── Proxmox Node: steven
    │     └── Primary workstation node
    │
    └── Proxmox Node: lab2
          ├── K8s Load Balancer VM
          ├── K8s Control Plane x3
          ├── K8s Worker x3
          ├── PostgreSQL VM (Ubuntu — Pickems database)
          ├── Ansible Controller VM
          └── LXCs: WireGuard · Pi-hole · Homarr · Uptime Kuma

Kubernetes Cluster (lab2)
    ├── ArgoCD          (GitOps)
    ├── Prometheus      (Metrics)
    ├── Grafana         (Dashboards)
    ├── Loki            (Logs)
    ├── Alertmanager    (Alerts)
    ├── Longhorn        (Storage)
    ├── Harbor          (Registry)
    ├── Vault           (Secrets)
    └── Sunday Pickems  (Application)
```

---

## 📋 Runbooks

Operational runbooks documenting common procedures and troubleshooting:

- [Bootstrap K3s Cluster](docs/runbooks/bootstrap-k3s-cluster.md)
- [Scheduler Not Running](docs/runbooks/scheduler-not-running.md)

---

## 🗺️ Roadmap

See the full [project roadmap](docs/roadmap/README.md) for planned improvements including:

- CI/CD pipeline — GitHub Actions → Harbor → ArgoCD auto-deploy
- Ansible dynamic inventory via Proxmox API
- Enterprise patching strategy
- AWS DR site with Terraform
- Sunday Pickems public launch (August 2026 — NFL season)
- AI features — weekly recap generator, pick suggestions

---

## 🛠️ Skills Demonstrated

- **Infrastructure as Code** — Terraform for VM/LXC provisioning, Ansible for configuration management
- **Kubernetes** — cluster bootstrapping, GitOps, persistent storage, ingress, observability
- **GitOps** — ArgoCD with GitHub as source of truth, app-of-apps pattern
- **Observability** — full stack metrics, logs, alerting with Prometheus/Grafana/Loki
- **Security** — secret management with Vault, VPN with WireGuard, private registry with Harbor
- **SaaS Development** — multi-tenant Flask app, RBAC, production deployment on Render
- **Networking** — DNS management, VPN, NAT, internal service discovery
- **Automation** — CI/CD pipelines, automated VM provisioning, cloud-init templates

---

## 📬 Contact

**Steven Barnes**

[LinkedIn](https://www.linkedin.com/in/steven-barnes-5b986856/) · [GitHub](https://github.com/stevenbarnes2017) · [sundaypickems.com](https://sundaypickems.com)
