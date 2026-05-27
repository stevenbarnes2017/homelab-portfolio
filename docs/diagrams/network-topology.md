# Network Topology Diagram

```mermaid
graph TB
    subgraph "Datacenter Network - 10.0.0.0/24"
        subgraph "Proxmox Host: lab2 (10.0.0.232)"
            subgraph "Kubernetes Cluster"
                CP1[k3s-cp-01<br/>10.0.0.120<br/>VMID 105]
                CP2[k3s-cp-02<br/>10.0.0.121<br/>VMID 108]
                CP3[k3s-cp-03<br/>10.0.0.122<br/>VMID 107]
                WK1[k3s-wk-01<br/>10.0.0.123<br/>VMID 109]
                WK2[k3s-wk-02<br/>10.0.0.124<br/>VMID 111]
                WK3[k3s-wk-03<br/>10.0.0.125<br/>VMID 112]
            end
            LB[k3s-lb-01<br/>10.0.0.119 MetalLB VIP<br/>VMID 119]
        end

        subgraph "Proxmox Host: steven (10.0.0.237)"
            DB[(FooballPoolDB01<br/>PostgreSQL<br/>VMID 102)]
            ANSIBLE[ansible-control<br/>Automation<br/>VMID 104]
        end

        PIHOLE[Pi-hole DNS<br/>*.lab.local]
        INTERNET[Internet]
    end

    %% Connections
    CP1 & CP2 & CP3 --> LB
    WK1 & WK2 & WK3 --> LB
    LB --> PIHOLE
    LB --> INTERNET
    DB -.-> WK2
    ANSIBLE -.-> CP1
    ANSIBLE -.-> CP2
    ANSIBLE -.-> CP3
    ANSIBLE -.-> WK1
    ANSIBLE -.-> WK2
    ANSIBLE -.-> WK3

    %% Styling
    classDef controlPlane fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef worker fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef infrastructure fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef external fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px

    class CP1,CP2,CP3 controlPlane
    class WK1,WK2,WK3 worker
    class LB,DB,ANSIBLE infrastructure
    class PIHOLE,INTERNET external
```

## Network Details

**Network Subnet:** 10.0.0.0/24  
**Gateway:** 10.0.0.1 (assumed)  
**DNS:** Pi-hole (resolves *.lab.local domains)

### IP Allocation

| Range | Purpose |
|-------|---------|
| 10.0.0.120-122 | Control Plane Nodes |
| 10.0.0.123-125 | Worker Nodes |
| 10.0.0.119 | MetalLB LoadBalancer VIP (Traefik) |
| 10.0.0.232 | Proxmox Host "lab2" |
| 10.0.0.237 | Proxmox Host "steven" |

### Service Domains (via MetalLB VIP 10.0.0.119)

All services accessible through Traefik ingress at 10.0.0.119:

- `argocd.lab.local` → ArgoCD GitOps
- `grafana.lab.local` → Grafana Dashboards
- `prometheus.lab.local` → Prometheus Metrics
- `alertmanager.lab.local` → Alert Manager
- `longhorn.lab.local` → Longhorn Storage UI
- `vault.lab.local` → HashiCorp Vault
- `hub.lab.local` → Harbor Registry
- `football.lab.local` → Sunday Pickems App
- `immich.lab.local` → Immich Photos

### Pod Network (Internal)

**CIDR:** 10.42.0.0/16 (k3s default)

| Node | Pod Network |
|------|-------------|
| k3s-cp-01 | 10.42.0.x |
| k3s-cp-02 | 10.42.2.x |
| k3s-cp-03 | 10.42.1.x |
| k3s-wk-01 | 10.42.5.x |
| k3s-wk-02 | 10.42.3.x |
| k3s-wk-03 | 10.42.4.x |

### Service Network (Internal)

**CIDR:** 10.43.0.0/16 (k3s default ClusterIP range)

### Network Flow

1. **External Access:** Internet → MetalLB VIP (10.0.0.119) → Traefik → Services
2. **Internal DNS:** Pi-hole resolves *.lab.local → 10.0.0.119
3. **Database Connection:** k8s pods → 10.0.0.x (FooballPoolDB01 on steven)
4. **Automation:** ansible-control → SSH → all k3s nodes
