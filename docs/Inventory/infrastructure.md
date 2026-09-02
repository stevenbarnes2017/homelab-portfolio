# Home Lab Infrastructure Inventory

## Purpose

This document records the currently verified home lab infrastructure.

The goal is to maintain a discoverable, version-controlled source of truth rather than relying entirely on manually maintained inventories.

Last verified: 2026-09-01

---

## Network

Primary LAN:

- Network: `10.0.0.0/24`
- Default Gateway: `10.0.0.1`
- Internet Provider: Xfinity
- Internet Edge: Xfinity ISP Gateway

Known configured inbound IPv4 port forwards:

| Protocol | Port | Destination | Purpose | Classification |
|---|---:|---|---|---|
| UDP | 51820 | 10.0.0.18 | WireGuard VPN | REQUIRED |

No other manual port-forwarding rules were identified in the Xfinity configuration.

In particular, public HTTP/HTTPS applications do not depend on manual TCP 80/443 forwarding. They use outbound Cloudflare Tunnel connections documented in `docs/security/cloudflare-ingress.md`.

IPv6 is enabled on multiple systems. Actual Internet reachability over IPv6 has not yet been externally verified.

---

## Proxmox Cluster

Both Proxmox hosts are members of the same Proxmox cluster.

| Host | IPv4 | Proxmox Version |
|---|---|---|
| lab2 | 10.0.0.232 | 9.2.11 |
| steven | 10.0.0.237 | 9.2.11 |

Both hosts currently run kernel:

`7.0.14-12-pve`

### Current Guest Placement

#### lab2

| ID | Type | Name | IPv4 / Role |
|---:|---|---|---|
| 102 | VM | FooballPoolDB01 | 10.0.0.129 |
| 105 | VM | k3s-cp-01 | 10.0.0.120 |
| 107 | VM | k3s-cp-03 | 10.0.0.122 |
| 108 | VM | k3s-cp-02 | 10.0.0.121 |
| 109 | VM | k3s-wk-01 | 10.0.0.123 |
| 111 | VM | k3s-wk-02 | 10.0.0.124 |
| 112 | VM | k3s-wk-03 | 10.0.0.125 |
| 119 | VM | k3s-lb-01 | 10.0.0.119 |
| 113 | LXC | wireguard | 10.0.0.18 |
| 150 | LXC | dns01 | 10.0.0.50 |

#### steven

| ID | Type | Name |
|---:|---|---|
| 100 | LXC | samba |
| 104 | VM | ansible-control |

Additional templates and test VMs exist but are not considered production infrastructure.

---

## WireGuard

WireGuard runs in LXC 113.

LAN interface:

- Hostname: `wireguard`
- IPv4: `10.0.0.18/24`
- VPN interface: `10.66.66.1/24`
- WireGuard port: UDP `51820`
- IPv4 forwarding: enabled
- IPv6 forwarding: disabled

Configured peer addresses:

- `10.66.66.2/32`
- `10.66.66.3/32`

WireGuard traffic is NATed through the LXC `eth0` interface.

The container currently has permissive IPv4 and IPv6 host firewall policies.

---

## DNS

DNS LXC:

- Hostname: `dns01`
- IPv4: `10.0.0.50/24`
- Gateway: `10.0.0.1`

Further DNS configuration will be documented separately.

---

## Local AI Backend

The Hermes user interface runs as Open WebUI in Kubernetes, while its Ollama model backend runs separately on a Windows home PC to use the system's NVIDIA RTX 2070 SUPER GPU and VRAM.

Verified Ollama network state:

- Host platform: Windows home PC
- Observed LAN IPv4: `10.0.0.41`
- Listener: `0.0.0.0:11434`
- Consumer: Open WebUI in the Kubernetes cluster, communicating across the LAN

Installed models observed:

- `interstellarninja/hermes-3-llama-3.1-8b-tools:latest`
- `hermes3:8b`

No credentials or model-service tokens are recorded in this inventory.

---

## Automation

Primary automation controller:

- VM: `ansible-control`
- Platform: Proxmox
- Role: Ansible control node

Existing automation includes:

- Ansible
- Terraform
- Kubernetes GitOps
- ArgoCD

Future inventory management should use infrastructure discovery and metadata rather than relying entirely on static Ansible inventories.

---

## Source of Truth Strategy

Long-term inventory sources should include:

- Proxmox API
- Kubernetes API
- Git
- Ansible
- monitoring systems
- future cloud APIs

This repository documents verified infrastructure state but should increasingly be generated or validated from live infrastructure APIs.
