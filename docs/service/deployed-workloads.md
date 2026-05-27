# Deployed Workloads & Services

## Overview

This document provides a comprehensive inventory of all services, applications, and infrastructure components deployed in the homelab Kubernetes cluster.

**Last Updated:** May 26, 2026
**Cluster Version:** k3s v1.34.6+k3s1 (primary), v1.35.4+k3s1 (wk-03)

---

## Namespace Summary

| Namespace | Purpose | Deployed Resources |
|-----------|---------|-------------------|
| `default` | Application workloads | Football app, nginx-test |
| `argocd` | GitOps continuous deployment | ArgoCD platform |
| `cert-manager` | Certificate management | cert-manager |
| `harbor` | Container registry | Private Docker registry |
| `immich` | Photo management | Immich photo server |
| `kube-system` | Core Kubernetes services | DNS, metrics, ingress |
| `longhorn-system` | Distributed storage | Longhorn storage platform |
| `metallb-system` | Load balancing | MetalLB |
| `monitoring` | Observability (deprecated) | Migrated to prometheus namespace |
| `prometheus` | Metrics & logging | Prometheus, Grafana, Loki, Alertmanager |
| `publice` | Public services | PostgreSQL exporter |
| `vault` | Secret management | HashiCorp Vault |
| `velero` | Backup & disaster recovery | Velero |

**Total Namespaces:** 15

---

## Application Services

### 🏈 Sunday Pickems (Football App)

**Namespace:** `default`  
**Domain:** `football.lab.local`  
**Status:** ✅ Running

**Components:**
- **football-web** (Deployment)
  - Image: Flask + Gunicorn web application
  - Replicas: 1
  - Running on: k3s-wk-02
  - Service: ClusterIP (port 80)
  
- **football-scheduler** (Deployment)
  - Image: APScheduler background worker
  - Replicas: 1
  - Running on: k3s-wk-02
  - Purpose: Automated weekly tasks (schedule imports, email reminders, scoring)

**Ingress:** HTTPS via Traefik (10.0.0.119)

**Observability:**
- Exposes `/metrics` endpoint
- Logs collected by Promtail → Loki
- Custom metric: `football_current_nfl_week`

---

### 📸 Immich (Photo Management)

**Namespace:** `immich`  
**Domain:** `immich.lab.local`  
**Status:** ✅ Running  
**Age:** 19 days

**Components:**
- **immich-server** (Deployment)
  - Main application server
  - Replicas: 1
  - Running on: k3s-wk-03
  
- **immich-machine-learning** (Deployment)
  - ML processing for photo recognition
  - Replicas: 1
  - Running on: k3s-wk-02
  
- **immich-valkey** (Deployment)
  - Redis-compatible cache
  - Replicas: 1
  - Running on: k3s-wk-01
  
- **cloudflared** (Deployment)
  - Cloudflare tunnel for external access
  - Replicas: 2
  - Running on: k3s-wk-02, k3s-cp-02

**Storage:**
- PVC: `immich-library` - 500 GB (Longhorn)
- Redis PVC: 2 GB

**Helm Chart:** immich-0.11.1 (app v2.6.3)

---

## Infrastructure Services

### ☸️ ArgoCD (GitOps)

**Namespace:** `argocd`  
**Domain:** `argocd.lab.local`  
**Status:** ⚠️ Mostly Running (repo-server degraded)  
**Age:** 48 days

**Components:**
- **argocd-server** - Web UI and API (✅ Running on cp-03)
- **argocd-application-controller** - App sync controller (✅ Running on cp-02)
- **argocd-repo-server** - Git repo interface (❌ Unknown state on wk-02)
- **argocd-redis** - Cache (✅ Running on cp-03)
- **argocd-dex-server** - SSO/OIDC (✅ Running on cp-03)
- **argocd-notifications-controller** - Notifications (✅ Running on cp-03)
- **argocd-applicationset-controller** - ApplicationSet CRD (✅ Running on cp-03)

**Purpose:** GitOps-based continuous deployment for cluster workloads

**Known Issue:** `argocd-repo-server` pod in Unknown state - needs investigation

---

### 🐳 Harbor (Container Registry)

**Namespace:** `harbor`  
**Domain:** `hub.lab.local`  
**Status:** ⚠️ Mostly Running  
**Age:** 46 days

**Components:**
- **harbor-core** - Main API and web UI (✅ Running on wk-03)
- **harbor-portal** - Web frontend (✅ Running on cp-03)
- **harbor-registry** - Docker registry (⚠️ 1/2 replicas running)
- **harbor-jobservice** - Background jobs (✅ Running on wk-02)
- **harbor-database** - PostgreSQL (✅ Running on wk-02)
- **harbor-redis** - Cache (✅ Running on wk-03)

**Storage:**
- Registry data: 5 GB (Longhorn)
- Database: 1 GB (Longhorn)
- Jobservice: 1 GB (Longhorn)
- Redis: 1 GB (Longhorn)
- Trivy scanner: 5 GB (Longhorn)

**Purpose:** Private container image registry for CI/CD pipeline

**Known Issue:** One registry replica stuck in ContainerCreating

---

### 🔐 HashiCorp Vault

**Namespace:** `vault`  
**Domain:** `vault.lab.local`  
**Status:** ❌ Degraded  
**Age:** 47 days

**Components:**
- **vault-0** (StatefulSet) - ❌ Stuck in ContainerCreating
- **vault-agent-injector** - ✅ Running on cp-01

**Storage:**
- PVC: 5 GB (Longhorn)

**Purpose:** Secret management and injection into Kubernetes workloads

**Helm Chart:** vault-0.32.0 (app v1.21.2)

**Critical Issue:** Main Vault pod not starting - blocking secret injection functionality

---

### 📦 Longhorn (Storage)

**Namespace:** `longhorn-system`  
**Domain:** `longhorn.lab.local`  
**Status:** ✅ Running  
**Age:** 48 days

**Components:**
- **longhorn-manager** (DaemonSet) - Storage management on all nodes (6/6 ready)
- **longhorn-driver-deployer** - CSI driver deployment
- **longhorn-ui** - Web dashboard (2 replicas)
- **csi-attacher, csi-provisioner, csi-resizer, csi-snapshotter** - CSI plugins
- **longhorn-csi-plugin** (DaemonSet) - CSI driver on all nodes

**Storage Class:** `longhorn` (default)

**Total Volumes Managed:** 9 PVs (totaling ~540 GB)

**Purpose:** Distributed block storage with replication and snapshots

---

### 🔀 MetalLB (Load Balancer)

**Namespace:** `metallb-system`  
**Status:** ✅ Running  
**Age:** 48 days

**Components:**
- **controller** - Load balancer controller (✅ Running on cp-02)
- **speaker** (DaemonSet) - L2 advertisement on all nodes (6/6 ready)

**VIP Assigned:** 10.0.0.119 (for Traefik ingress)

**Purpose:** Bare-metal load balancer for service type LoadBalancer

---

### 🌐 Traefik (Ingress Controller)

**Namespace:** `kube-system`  
**Status:** ✅ Running  
**Age:** 48 days

**Components:**
- **traefik** (Deployment) - Ingress controller (✅ Running on cp-01)
- **svclb-traefik** (DaemonSet) - Service load balancer helpers (6/6 ready)

**LoadBalancer IP:** 10.0.0.119 (via MetalLB)

**Helm Chart:** traefik-39.0.501+up39.0.5 (app v3.6.10)

**Ingress Routes Managed:** 10 domains (*.lab.local)

**Purpose:** HTTP/HTTPS ingress routing and TLS termination

---

### 🔒 cert-manager

**Namespace:** `cert-manager`  
**Status:** ✅ Running  
**Age:** 38 days

**Components:**
- **cert-manager** - Certificate controller (✅ Running on cp-03)
- **cert-manager-webhook** - Webhook server (✅ Running on cp-03)
- **cert-manager-cainjector** - CA injection (✅ Running on cp-02)

**Helm Chart:** cert-manager-v1.20.2 (app v1.20.2)

**Purpose:** Automated TLS certificate management for internal services

**Issuer Type:** Self-signed ClusterIssuer for *.lab.local domains

---

## Observability Stack

### 📊 Prometheus Stack

**Namespace:** `prometheus`  
**Status:** ✅ Running  
**Age:** 47 days

**Components:**

#### Prometheus
- **prometheus-kube-prometheus-stack-prometheus** (StatefulSet)
- Replicas: 1 (Running on wk-01)
- Storage: 10 GB PVC (Longhorn)
- Domain: `prometheus.lab.local`
- Purpose: Metrics collection and querying

#### Grafana
- **kube-prometheus-stack-grafana** (Deployment)
- Replicas: 1 (Running on cp-03)
- Domain: `grafana.lab.local`
- Purpose: Metrics visualization and dashboards

#### Alertmanager
- **alertmanager-kube-prometheus-stack-alertmanager** (StatefulSet)
- Replicas: 1 (Running on cp-01)
- Domain: `alertmanager.lab.local`
- Purpose: Alert routing and notifications

#### Exporters
- **prometheus-node-exporter** (DaemonSet) - System metrics on all 6 nodes
- **kube-state-metrics** - Kubernetes object metrics
- **proxmox-exporter** - Proxmox host metrics

**Total ConfigMaps:** 33  
**Total Secrets:** 15

---

### 📝 Loki (Logging)

**Namespace:** `prometheus`  
**Status:** ✅ Running  
**Age:** 45 days

**Components:**
- **loki** (StatefulSet) - Log aggregation server (✅ Running on cp-02)
- **loki-promtail** (DaemonSet) - Log collection agent on all 6 nodes

**Purpose:** Centralized log aggregation from all cluster workloads

**Integration:** Logs viewable in Grafana dashboards

---

## Backup & Disaster Recovery

### 💾 Velero

**Namespace:** `velero`  
**Status:** ✅ Running  
**Age:** 10 days

**Components:**
- **velero** (Deployment) - Backup controller (✅ Running on wk-02)
- **node-agent** (DaemonSet) - Backup agent on all 6 nodes

**Purpose:** Kubernetes cluster backup and disaster recovery

**Integration:** Backs up Longhorn volumes and Kubernetes resources

---

## Supporting Services

### Core Kubernetes Services

**Namespace:** `kube-system`

- **coredns** - DNS resolution for cluster services
- **metrics-server** - Resource metrics API
- **local-path-provisioner** - Local storage provisioner

---

### Test/Debug Workloads

**Namespace:** `default`

- **nginx-test** - Test web server (NodePort 31733)
- **node-debugger-k3s-wk-03** - Debug pods (Completed)

---

## Service Domains (*.lab.local)

| Service | Domain | Ingress Class | TLS |
|---------|--------|---------------|-----|
| ArgoCD | argocd.lab.local | traefik | ✅ |
| Grafana | grafana.lab.local | traefik | ✅ |
| Prometheus | prometheus.lab.local | traefik | ✅ |
| Alertmanager | alertmanager.lab.local | traefik | ✅ |
| Longhorn | longhorn.lab.local | traefik | ✅ |
| Vault | vault.lab.local | traefik | ✅ |
| Harbor | hub.lab.local | traefik | ❌ (HTTP only) |
| Football App | football.lab.local | traefik | ✅ |
| Immich | immich.lab.local | traefik | ✅ |
| nginx-test | NodePort 31733 | - | - |

**All domains resolve via Pi-hole to:** 10.0.0.119 (MetalLB VIP)

---

## Resource Summary

### Deployments
- Total: 37 deployments across all namespaces
- Healthy: 35
- Degraded: 2 (argocd-repo-server, vault)

### StatefulSets
- Total: 7 statefulsets
- Running: 6
- Degraded: 1 (vault-0)

### DaemonSets
- Total: 8 daemonsets
- All running on 6/6 nodes

### Helm Releases
- Total: 6 Helm charts deployed
- cert-manager, immich, immich-redis, traefik, traefik-crd, vault

### Persistent Volumes
- Total PVs: 9
- Total Capacity: ~540 GB
- All volumes: Bound
- Storage Class: Longhorn (default)

---

## Known Issues & Action Items

### Critical Issues
1. **Vault pod stuck in ContainerCreating** - Blocking secret injection
2. **ArgoCD repo-server in Unknown state** - Affecting GitOps sync
3. **Harbor registry replica not starting** - Reduced redundancy

### Warnings
1. **Mixed k3s versions** - wk-03 on v1.35.4, others on v1.34.6
2. **monitoring namespace unused** - Cleanup needed
3. **Some pods with high restart counts** - Investigate stability

### Improvements Needed
1. Standardize k3s versions across cluster
2. Document Grafana dashboards
3. Export Prometheus alert rules
4. Configure Velero backup schedule
5. Implement proper health checks for all services

---

## CI/CD Pipeline

**Image Build:** GitHub Actions  
**Image Registry:** Harbor (hub.lab.local)  
**Deployment:** Manual kubectl apply (ArgoCD not fully utilized yet)  
**Image Tag Strategy:** `latest` + version tags

---

## Future Service Additions

Planned deployments from roadmap:
- SMS notification service (Twilio integration)
- External uptime monitoring
- Additional exporters for enhanced observability
- Backup storage target (external to cluster)
