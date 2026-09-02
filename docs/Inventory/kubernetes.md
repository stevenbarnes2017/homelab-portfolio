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

K3s uses Flannel networking. The `flannel.1` and `cni0` interfaces were verified on `k3s-wk-01`.

Observed pod-network interfaces include:

- `flannel.1`
- `cni0`

Verified node pod CIDRs:

| Node | Pod CIDR |
|---|---|
| k3s-cp-01 | `10.42.0.0/24` |
| k3s-cp-03 | `10.42.1.0/24` |
| k3s-cp-02 | `10.42.2.0/24` |
| k3s-wk-02 | `10.42.3.0/24` |
| k3s-wk-03 | `10.42.4.0/24` |
| k3s-wk-01 | `10.42.5.0/24` |

Service ClusterIP addresses use:

`10.43.0.0/16`

### NetworkPolicy Enforcement and Current Coverage

K3s kube-router NetworkPolicy enforcement is active. `KUBE-ROUTER-*` and `KUBE-POD-FW-*` iptables chains were verified.

Current policy coverage includes ArgoCD, Immich Redis, and the Celestial and Travel Planner segmentation pilots. Most namespaces still have no NetworkPolicy and therefore remain default-allow for east-west traffic.

Verified policy observations:

- Immich Redis permits ingress on TCP `6379` without a source selector, so any reachable source may connect on that port. Its egress is unrestricted.
- ArgoCD Redis is more tightly restricted to specific ArgoCD components.
- Several ArgoCD policies use `namespaceSelector: {}`, allowing ingress from any namespace.
- The policy selecting `argocd-server` contains `ingress: - {}`, which is effectively unrestricted ingress to the selected pod.

No cluster-wide blanket default-deny policy has been implemented. Celestial and Travel Planner now have verified workload-specific default-deny and explicit-allow models.

### Celestial NetworkPolicy Pilot

The `celestial` namespace has NetworkPolicy enforcement through four policies stored in `kubernetes/celestial/networkpolicy.yaml` in the infrastructure Git repository:

- `celestial-default-deny-ingress`
- `celestial-allow-ingress`
- `celestial-default-deny-egress`
- `celestial-allow-required-egress`

The existing `argocd-app.yaml`, `deployment.yaml`, `namespace.yaml`, and `service-ingress.yaml` manifests remain unchanged alongside the policy manifest. ArgoCD automatically synchronized the NetworkPolicy after it was committed to Git.

Ingress is default-deny. Traffic from the Cloudflare Tunnel (`cloudflared`) pods is explicitly permitted, while an unrelated Kubernetes pod cannot directly connect to Celestial. A Traefik allowance remains for the unused `celestial.lab.local` ingress and is a cleanup candidate, not a required production path.

Egress is default-deny with explicit allowances for:

- CoreDNS over TCP and UDP `53`
- PostgreSQL at `10.0.0.129:5432`
- External HTTPS APIs over TCP `443`

The generic Internet HTTPS allowance excludes private RFC1918 networks unless a destination is explicitly permitted. This limits lateral movement from a compromised Celestial workload.

The verified production request path is:

```text
https://celestial-engine.vercel.app
-> Vercel frontend
-> https://celestial-api.sundaypickems.com
-> Cloudflare
-> Cloudflare Tunnel
-> Celestial Kubernetes Service
-> Celestial API pod
```

Validation performed:

1. Kubernetes server-side dry run succeeded for all four NetworkPolicy resources.
2. ArgoCD synchronized the policies successfully.
3. `kubectl get networkpolicy -n celestial` confirmed all four policies were active.
4. `curl -i https://celestial-api.sundaypickems.com/health` returned HTTP `200` with `{"engine":"Celestial Engine API v1.0","global_free_beta":true,"status":"healthy"}`.
5. In the negative east-west test, a temporary curl pod in `default` resolved `celestial-api.celestial.svc.cluster.local` to `10.43.234.199`, but its direct connection failed with `Connection refused`; the legitimate Cloudflare path remained operational.

### Travel Planner NetworkPolicy Pilot

The second successfully segmented pilot is the `travel-planner-api` workload in namespace `travel-planner`, selected by `app=travel-planner-api`. The `travel-planner-api` Service maps port `80` to pod port `8000`; the observed backend endpoint was `10.42.4.157:8000`.

Four NetworkPolicies are active:

- `travel-planner-default-deny-ingress`
- `travel-planner-allow-ingress`
- `travel-planner-default-deny-egress`
- `travel-planner-allow-required-egress`

Ingress is default-deny. `cloudflared` in namespace `immich`, selected by `app=cloudflared`, is explicitly allowed to TCP `8000`. Traefik in `kube-system` is also currently allowed to TCP `8000`, although `travel.lab.local` is not an actively used frontend path; this allowance is a possible cleanup item rather than a verified production requirement. Arbitrary east-west traffic is not allowed.

Egress is default-deny with explicit allowances for CoreDNS over TCP and UDP `53`, PostgreSQL at `10.0.0.129:5432`, and public HTTPS over TCP `443`. The general HTTPS rule excludes RFC1918 networks.

The verified production request path is:

```text
https://travel-mobile-app-lime.vercel.app/
-> Vercel frontend
-> travel-app.sundaypickems.com
-> Cloudflare
-> Cloudflare Tunnel / cloudflared
-> travel-planner-api Service
-> travel-planner-api pod
```

Validation performed:

1. `kubectl apply --dry-run=server` successfully validated all four policies.
2. ArgoCD deployed and reconciled the policies from the GitOps repository.
3. `https://travel-app.sundaypickems.com/health` returned HTTP/2 `200` with `{"status":"ok"}` after the correct `cloudflared` allowance was committed.
4. An arbitrary pod in namespace `default` resolved the service DNS and ClusterIP, but its TCP connection was refused. This demonstrated that DNS remained available while unauthorized application connectivity was blocked.

During troubleshooting, a corrected NetworkPolicy was applied manually with `kubectl`, but ArgoCD restored the older version because Git remained the source of truth. After commit `047e80f` (`fix travel planner cloudflare ingress`) added the corrected `cloudflared` rule to Git, ArgoCD reconciled the intended configuration and the public endpoint returned `200`. Persistent GitOps changes must therefore be committed to Git rather than applied only with `kubectl`.

### Segmentation Dependencies

Any future default-deny or egress design must preserve required flows, including:

- Open WebUI in namespace `ai` to Ollama at `10.0.0.41:11434`
- Kubernetes workloads to external-LAN PostgreSQL dependencies at `10.0.0.129`
- `cloudflared` in namespace `immich` to published application services across namespaces
- Prometheus cross-namespace scraping
- DNS access to CoreDNS in `kube-system`

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
