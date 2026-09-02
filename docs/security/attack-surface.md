# Internet Attack Surface

## Objective

Document every path by which Internet-originated traffic may reach the home lab.

This document represents verified current state and known unknowns.

Last verified: 2026-09-01

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

There are no verified manual TCP 80/443 port forwards. Public web applications use Cloudflare Tunnel and must not be represented as entering through Xfinity 80/443 forwarding or the Traefik LAN address.

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

---

## Verified Public Web Path

```text
Internet
   |
   v
Cloudflare
   |
   v
Cloudflare Tunnel
   |
   v
Kubernetes ClusterIP service
   |
   v
Application pod
```

Six public hostnames and their current service destinations are documented in `cloudflare-ingress.md`. The path is carried by outbound connections from the `cloudflared` deployment in Kubernetes.

Traefik remains the LAN ingress controller and only discovered Kubernetes `LoadBalancer`, but it is not the verified public HTTP/HTTPS edge path.

---

## Hermes / Ollama Path

Hermes/Open WebUI is Internet-accessible at `hermes.barnesfamily-pics.online` through Cloudflare Tunnel. The tunnel routes to `open-webui.ai.svc.cluster.local:8080`.

Open WebUI then communicates across the LAN with Ollama on the Windows home PC at the observed address `10.0.0.41:11434`. Ollama currently binds to `0.0.0.0`, so its API listener is exposed to networks permitted to reach that host. This is an assessment observation and does not by itself establish Internet reachability.
