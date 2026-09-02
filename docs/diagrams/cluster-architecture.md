# Kubernetes Cluster Architecture

```mermaid
graph TB
    subgraph "Ingress Layer"
        INTERNET[Internet/Users]
        CLOUDFLARE[Cloudflare Edge<br/>Tunnel]
        CLOUDFLARED[cloudflared x2<br/>immich namespace]
        PUBLICSVC[Public application<br/>ClusterIP services]
        METALLB[MetalLB<br/>10.0.0.119]
        TRAEFIK[Traefik LAN Ingress<br/>LoadBalancer 80/443]
    end

    subgraph "Control Plane - HA Configuration"
        CP1[k3s-cp-01<br/>4GB RAM, 2 vCPU]
        CP2[k3s-cp-02<br/>4GB RAM, 2 vCPU]
        CP3[k3s-cp-03<br/>4GB RAM, 2 vCPU]
        ETCD[(etcd<br/>distributed)]
    end

    subgraph "Worker Nodes"
        WK1[k3s-wk-01<br/>4GB RAM, 2 vCPU<br/>100GB disk]
        WK2[k3s-wk-02<br/>6GB RAM, 2 vCPU<br/>90GB disk]
        WK3[k3s-wk-03<br/>4GB RAM, 2 vCPU<br/>88.5GB disk]
    end

    subgraph "Pod Network and Policy"
        FLANNEL[Flannel Data Plane<br/>10.42.0.0/16]
        KUBEROUTER[kube-router<br/>NetworkPolicy Enforcement]
        DEFAULTALLOW[Current State<br/>Mostly Default-Allow]
    end

    subgraph "Storage Layer"
        LONGHORN[Longhorn Distributed Storage<br/>540GB managed volumes]
        EXTERNAL[(Immich External Storage<br/>916GB dedicated disk)]
    end

    subgraph "Observability Stack"
        PROM[Prometheus<br/>Metrics Collection]
        GRAFANA[Grafana<br/>Visualization]
        LOKI[Loki<br/>Log Aggregation]
        ALERT[Alertmanager<br/>Notifications]
    end

    subgraph "Platform Services"
        ARGOCD[ArgoCD<br/>GitOps]
        HARBOR[Harbor<br/>Registry]
        VAULT[Vault<br/>Secrets]
        CERT[cert-manager<br/>TLS]
    end

    subgraph "Application Workloads"
        FOOTBALL[Sunday Pickems<br/>Flask App]
        IMMICH[Immich<br/>Photos]
        SCHEDULER[Football Scheduler<br/>Background Jobs]
        OPENWEBUI[Hermes / Open WebUI]
    end

    subgraph "External Dependencies"
        DB[(PostgreSQL<br/>on steven host)]
        ANSIBLE[Ansible Controller<br/>on steven host]
        OLLAMA[Ollama<br/>Windows PC 10.0.0.41:11434]
    end

    %% Ingress Flow
    INTERNET --> CLOUDFLARE
    CLOUDFLARE --> CLOUDFLARED
    CLOUDFLARED --> PUBLICSVC
    PUBLICSVC --> FOOTBALL
    PUBLICSVC --> IMMICH
    PUBLICSVC --> OPENWEBUI
    METALLB --> TRAEFIK
    TRAEFIK --> FOOTBALL
    TRAEFIK --> IMMICH
    TRAEFIK --> GRAFANA
    TRAEFIK --> ARGOCD
    TRAEFIK --> HARBOR

    %% Control Plane
    CP1 & CP2 & CP3 --> ETCD
    TRAEFIK --> CP1 & CP2 & CP3

    %% Workers
    FOOTBALL --> WK2
    IMMICH --> WK2
    SCHEDULER --> WK2
    HARBOR --> WK2
    ARGOCD --> CP3
    PROM --> WK1
    GRAFANA --> CP3
    WK1 & WK2 & WK3 --> FLANNEL
    FLANNEL --> KUBEROUTER
    KUBEROUTER --> DEFAULTALLOW

    %% Storage
    WK1 & WK2 & WK3 --> LONGHORN
    IMMICH --> EXTERNAL

    %% Observability
    FOOTBALL & IMMICH & SCHEDULER -.metrics.-> PROM
    PROM --> GRAFANA
    LOKI --> GRAFANA
    PROM --> ALERT
    WK1 & WK2 & WK3 -.logs.-> LOKI

    %% Platform
    ARGOCD -.deploys.-> FOOTBALL
    ARGOCD -.deploys.-> IMMICH
    VAULT -.secrets.-> FOOTBALL
    CERT -.TLS.-> TRAEFIK

    %% External
    FOOTBALL --> DB
    ANSIBLE -.manages.-> CP1 & CP2 & CP3
    ANSIBLE -.manages.-> WK1 & WK2 & WK3
    OPENWEBUI --> OLLAMA

    %% Styling
    classDef controlPlane fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef worker fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef storage fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef observability fill:#e0f2f1,stroke:#004d40,stroke-width:2px
    classDef platform fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef application fill:#e8eaf6,stroke:#1a237e,stroke-width:2px
    classDef external fill:#f1f8e9,stroke:#33691e,stroke-width:2px
    classDef ingress fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef network fill:#ede7f6,stroke:#4527a0,stroke-width:2px

    class CP1,CP2,CP3,ETCD controlPlane
    class WK1,WK2,WK3 worker
    class LONGHORN,EXTERNAL storage
    class PROM,GRAFANA,LOKI,ALERT observability
    class ARGOCD,HARBOR,VAULT,CERT platform
    class FOOTBALL,IMMICH,SCHEDULER,OPENWEBUI application
    class DB,ANSIBLE,OLLAMA external
    class METALLB,TRAEFIK,INTERNET,CLOUDFLARE,CLOUDFLARED,PUBLICSVC ingress
    class FLANNEL,KUBEROUTER,DEFAULTALLOW network
```

## Architecture Highlights

### High Availability Design

**Control Plane:**
- 3 control plane nodes with embedded etcd
- Distributed consensus via etcd cluster
- MetalLB provides stable VIP for API access
- Any control plane node can handle requests

**Storage:**
- Longhorn distributed block storage across all 6 nodes
- Data replication for fault tolerance
- Automatic volume failover

**Application:**
- Multiple replicas for critical services
- Pod distribution across worker nodes
- Rolling updates for zero-downtime deployments

### Resource Distribution

**Control Plane (12 GB total):**
- Kubernetes API Server
- etcd distributed database
- Controller managers
- Scheduler
- ArgoCD application controller

**Workers (14 GB total):**
- Application workloads
- Storage backend (Longhorn)
- Observability agents
- Ingress controllers

**k3s-wk-02 (6 GB):**
- Harbor registry (heavy workload)
- Immich server (photo processing)
- Football web application
- Receives extra RAM allocation

### Data Flow Patterns

1. **User Request Flow:**
   ```
   User → Internet → Cloudflare → Cloudflare Tunnel → ClusterIP Service → Application Pod
   ```

   There are no verified manual TCP 80/443 port forwards. MetalLB and Traefik provide LAN ingress for internal routes; they are not the verified public web edge.

2. **Deployment Flow:**
   ```
   Git Push → GitHub Actions → Harbor Registry → ArgoCD → K8s Deployment
   ```

3. **Observability Flow:**
   ```
   Pods → Metrics Endpoint → Prometheus → Grafana
   Pods → Logs → Promtail → Loki → Grafana
   Prometheus → Alert Rules → Alertmanager → Email
   ```

4. **Secret Injection:**
   ```
   Vault → Agent Injector → Pod Environment Variables
   ```

### Network Isolation

- **Pod data plane:** Flannel, using node `/24` pod CIDRs within `10.42.0.0/16`
- **Policy enforcement:** kube-router is active; verified iptables chains include `KUBE-ROUTER-*` and `KUBE-POD-FW-*`
- **Current policy state:** mostly default-allow because most namespaces have no NetworkPolicy
- **Service Network:** 10.43.0.0/16 (ClusterIP)
- **Node Network:** 10.0.0.0/24 (physical)
- **Public web access:** Cloudflare Tunnel to selected ClusterIP services
- **LAN ingress:** Traefik via MetalLB VIP `10.0.0.119`
- **Management plane:** ArgoCD, Longhorn, Vault, Harbor, Prometheus, Grafana, and Alertmanager should be treated as management-plane services in future segmentation

No default-deny policy is shown as implemented. Any future segmentation must preserve Cloudflare Tunnel cross-namespace routes, Prometheus scraping, CoreDNS, Open WebUI-to-Ollama, and external-LAN PostgreSQL access.

### Storage Tiers

**Tier 1 - Critical Data:**
- Prometheus metrics (10 GB SSD-backed Longhorn)
- Vault secrets (5 GB encrypted Longhorn)
- Harbor registry (5 GB Longhorn)

**Tier 2 - Application Data:**
- Immich photos (500 GB dedicated external disk)
- Database (60 GB on separate VM)

**Tier 3 - Cache/Temporary:**
- Redis caches (2-5 GB Longhorn)
- Temporary pod storage (ephemeral)
