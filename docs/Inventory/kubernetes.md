# Kubernetes / K3s Inventory

## Overview

The home lab runs a highly available K3s cluster hosted on Proxmox VMs.

Last verified: 2026-09-01

K3s version:

`v1.36.2+k3s1`

Operating system:

`Ubuntu 24.04.4 LTS`

Container runtime:

`containerd 2.3.2-k3s2`

---

## Nodes

| Node | Role | IPv4 |
|---|---|---|
| k3s-lb-01 | Load Balancer | 10.0.0.119 |
| k3s-cp-01 | Control Plane / etcd | 10.0.0.120 |
| k3s-cp-02 | Control Plane / etcd | 10.0.0.121 |
| k3s-cp-03 | Control Plane / etcd | 10.0.0.122 |
| k3s-wk-01 | Worker | 10.0.0.123 |
| k3s-wk-02 | Worker | 10.0.0.124 |
| k3s-wk-03 | Worker | 10.0.0.125 |

All Kubernetes nodes were verified Ready on 2026-08-23.

---

## Pod Networking

K3s uses Flannel networking.

Observed pod-network interfaces include:

- `flannel.1`
- `cni0`

Observed pod CIDRs include networks within:

`10.42.0.0/16`

Service ClusterIP addresses use:

`10.43.0.0/16`

---

## Kubernetes Ingress

Traefik is the cluster ingress controller.

Service:

- Namespace: `kube-system`
- Name: `traefik`
- Type: `LoadBalancer`
- External IP: `10.0.0.119`

Ports:

- TCP 80
- TCP 443

Current Kubernetes service representation:

`80:32666/TCP`
`443:31711/TCP`

Traefik was the only Kubernetes `LoadBalancer` service discovered. Application services behind the ingress and Cloudflare Tunnel paths are predominantly `ClusterIP` services.

---

## Non-ClusterIP Services

Currently identified:

| Namespace | Service | Type | Exposure |
|---|---|---|---|
| kube-system | traefik | LoadBalancer | 10.0.0.119:80/443 |
| default | nginx-test | NodePort | TCP 31733 |

The `nginx-test` NodePort requires review to determine whether it is still needed.

---

## Current Ingress Hosts

### Internal / lab.local

- argocd.lab.local
- celestial.lab.local
- football.lab.local
- hub.lab.local
- homarr.lab.local
- immich.lab.local
- longhorn.lab.local
- lidarr.lab.local
- music.lab.local
- prowlarr.lab.local
- qbittorrent.lab.local
- radarr.lab.local
- slskd.lab.local
- sonarr.lab.local
- soularr.lab.local
- threadfin.lab.local
- whisparr.lab.local
- alertmanager.lab.local
- grafana.lab.local
- prometheus.lab.local
- travel.lab.local
- vault.lab.local
- vaultwarden.lab.local

### Public hostname also represented by Kubernetes ingress

- hermes.barnesfamily-pics.online

This hostname is verified as Internet-accessible through Cloudflare Tunnel to the Open WebUI `ClusterIP` service. Its public traffic does not depend on direct TCP 80/443 forwarding to Traefik.

---

## Cloudflare Tunnel

Verified deployment:

- Namespace: `immich`
- Deployment: `cloudflared`
- Replicas: `2`
- Authentication reference: Kubernetes Secret `cloudflare-tunnel-token`
- Tunnel ID: `ffd6a53d-120a-41e7-b9df-59458f534b17`
- Configuration management: remote Cloudflare configuration
- Ingress termination: final `http_status:404` catch-all

The detailed public hostname-to-service map is maintained in `docs/security/cloudflare-ingress.md`.

---

## Management-Plane Services

The following administrative applications should be treated as management-plane services when network segmentation and access policy are designed:

- ArgoCD
- Longhorn
- Vault
- Harbor
- Prometheus
- Grafana
- Alertmanager

---

## Core Platform Services

The cluster currently includes:

- ArgoCD
- Traefik
- MetalLB
- Longhorn
- Harbor
- Vault
- Vaultwarden
- Prometheus
- Grafana
- Loki
- Alertmanager
- cert-manager
- MinIO
- Immich
- media workloads
- AI workloads
- application workloads

---

## IPv6

Kubernetes VMs currently receive globally scoped IPv6 addresses in addition to IPv4.

Example prefix observed:

`2601:281:c900:c220::/64`

External inbound IPv6 reachability has not yet been verified.

This must be included in the attack-surface review.
