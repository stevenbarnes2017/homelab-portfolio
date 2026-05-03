# 🏠 Homelab & Platform Portfolio

### Steven Barnes · Infrastructure Engineer · Platform Engineer · SRE
### LinkedIn · Sunday Pickems


## Overview
This repository documents my homelab infrastructure, DevOps practices, and the Sunday Pickems platform — a real-world, multi-tenant SaaS application I designed, built, and operate end-to-end.
The lab serves as a personal learning environment built to enterprise standards — everything here reflects skills and practices directly applicable to production infrastructure engineering roles.

🚀 Projects
Sunday Pickems Platform
A production Flask web application for running NFL confidence-based pick'em pools. Multi-tenant architecture supporting multiple independent groups with role-based access control.

Live site: sundaypickems.com
Stack: Python/Flask, PostgreSQL, Docker, Kubernetes, Vault, Prometheus
Users: Real users, live NFL season data, automated email via Brevo
Full documentation →

Homelab Kubernetes Cluster
A production-grade Kubernetes cluster running on Proxmox — fully automated from bare metal to running applications using Terraform and Ansible.

Cluster: 1 load balancer, 3 control planes, 3 workers
GitOps: ArgoCD synced to GitHub as source of truth
Storage: Longhorn persistent storage
Observability: Prometheus, Grafana, Loki, Alertmanager
Security: HashiCorp Vault for secret management
Registry: Harbor private container registry
Full documentation →


🏗️ Infrastructure Stack
Compute & Virtualization
TechnologyRoleProxmox VEHypervisor — 2 node clusterUbuntu 22.04VM OS via cloud-init templatesDebian 12LXC container OS
Kubernetes & GitOps
TechnologyRoleKubernetesContainer orchestration — 7 node clusterArgoCDGitOps — GitHub as source of truthLonghornPersistent storageHarborPrivate container registryHashiCorp VaultSecret management
Observability
TechnologyRolePrometheusMetrics collectionGrafanaDashboards and visualizationLokiLog aggregationAlertmanagerAlert routingUptime KumaService uptime monitoring
Infrastructure as Code
TechnologyRoleTerraformVM and LXC provisioning on ProxmoxAnsibleConfiguration management, patchingGitHub ActionsCI/CD pipelines
Networking & Security
TechnologyRoleWireGuardVPN — secure remote access to homelabPi-holeLocal DNS resolverHomarrService dashboard
Application Stack
TechnologyRolePython / FlaskWeb application frameworkPostgreSQLPrimary databaseDockerContainerizationRenderProduction hostingBrevoTransactional email

📁 Repository Structure
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

🗺️ Architecture Overview
Internet
    │
    ▼
Xfinity Gateway (NAT)
    │
    ▼
Homelab Network (10.0.0.0/24)
    │
    ├── Proxmox Node 1 (lab1)
    │     ├── K8s Load Balancer VM
    │     ├── K8s Control Plane x3
    │     ├── K8s Worker x3
    │     └── LXCs: WireGuard, Pi-hole, Homarr
    │
    └── Proxmox Node 2 (lab2)
          ├── Ansible Controller VM
          └── Postgres SQL DB Server
          

Kubernetes Cluster
    ├── ArgoCD          (GitOps)
    ├── Prometheus      (Metrics)
    ├── Grafana         (Dashboards)
    ├── Loki            (Logs)
    ├── Alertmanager    (Alerts)
    ├── Longhorn        (Storage)
    ├── Harbor          (Registry)
    ├── Vault           (Secrets)
    └── Sunday Pickems  (Application)

📋 Runbooks
Operational runbooks documenting common procedures and troubleshooting guides:

Bootstrap K3s Cluster
Scheduler Not Running


🗺️ Roadmap
See the full project roadmap for planned improvements including:

CI/CD pipeline — GitHub Actions → Harbor → ArgoCD auto-deploy
Ansible dynamic inventory via Proxmox API
Enterprise patching strategy
AWS DR site with Terraform
Sunday Pickems public launch (August 2026 — NFL season)
AI features — weekly recap generator, pick suggestions


🛠️ Skills Demonstrated

Infrastructure as Code — Terraform for VM/LXC provisioning, Ansible for configuration management
Kubernetes — cluster bootstrapping, GitOps, persistent storage, ingress, observability
GitOps — ArgoCD with GitHub as source of truth, app-of-apps pattern
Observability — full stack metrics, logs, alerting with Prometheus/Grafana/Loki
Security — secret management with Vault, VPN with WireGuard, private registry with Harbor
SaaS Development — multi-tenant Flask app, RBAC, production deployment on Render
Networking — DNS management, VPN, NAT, internal service discovery
Automation — CI/CD pipelines, automated VM provisioning, cloud-init templates


📬 Contact
Steven Barnes
LinkedIn · sundaypickems.com
 
