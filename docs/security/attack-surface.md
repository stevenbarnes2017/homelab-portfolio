@'
# Internet Attack Surface

## Objective

Document every path by which Internet-originated traffic may reach the home lab.

This document represents verified current state and known unknowns.

Last verified: 2026-08-23

---

## Internet Edge

Internet provider:

Xfinity

Gateway:

Xfinity-managed ISP gateway

LAN:

`10.0.0.0/24`

---

## Verified IPv4 Port Forwarding

| Protocol | External Port | Internal Destination | Service | Classification |
|---|---:|---|---|---|
| UDP | 51820 | 10.0.0.18 | WireGuard | REQUIRED |

No additional manual IPv4 port forwards were identified.

---

## WireGuard Path

Internet
   |
   | UDP 51820
   v
Xfinity Gateway
   |
   v
10.0.0.18
WireGuard LXC
   |
   | wg0
   v
10.66.66.0/24
VPN Clients
   |
   v
Home Lab Network