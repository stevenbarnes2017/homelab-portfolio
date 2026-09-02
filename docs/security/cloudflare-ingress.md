# Cloudflare Internet Ingress

**Last verified:** 2026-09-01

## Overview

Public home-lab applications are exposed using Cloudflare Tunnel rather than inbound TCP 80/443 port forwarding on the Xfinity gateway.

The Cloudflare tunnel is currently named:

`immich-pictures`

The name predates its use as the shared tunnel for multiple applications.

## Architecture

```text
Internet
    |
    v
Cloudflare Edge
    |
    +-- Cloudflare Access where configured
    |
    v
Cloudflare Tunnel: immich-pictures
    |
    | Outbound tunnel connections
    v
cloudflared Deployment
Kubernetes namespace: immich
Replicas: 2
    |
    v
Kubernetes ClusterIP Services
```

This is the effective public web path. It is not `Internet -> Xfinity TCP 80/443 -> Traefik`; no manual TCP 80/443 port forwards were identified.

## Verified Tunnel Deployment

- Namespace: `immich`
- Deployment: `cloudflared`
- Replicas: `2`
- Authentication reference: Kubernetes Secret `cloudflare-tunnel-token`
- Tunnel ID: `ffd6a53d-120a-41e7-b9df-59458f534b17`
- Configuration: remotely managed by Cloudflare
- Final ingress rule: `http_status:404` catch-all

Secret values and tunnel credentials are intentionally excluded from this documentation.

## Verified Public Routes

| Public hostname | Kubernetes service destination |
|---|---|
| `www.barnesfamily-pics.online` | `http://immich-server.immich.svc.cluster.local:2283` |
| `sundaypickems.com` | `http://football-web.default.svc.cluster.local:80` |
| `celestial-api.sundaypickems.com` | `http://celestial-api.celestial.svc.cluster.local:80` |
| `travel-app.sundaypickems.com` | `http://travel-planner-api.travel-planner.svc.cluster.local:80` |
| `music.sundaypickems.com` | `http://navidrome.media.svc.cluster.local:80` |
| `hermes.barnesfamily-pics.online` | `http://open-webui.ai.svc.cluster.local:8080` |

## Hermes / Local LLM Architecture

`hermes.barnesfamily-pics.online` terminates through Cloudflare Tunnel at Open WebUI in the Kubernetes cluster. Open WebUI communicates across the LAN with Ollama on a Windows home PC at the observed address `10.0.0.41:11434`. Ollama runs on that system to use its NVIDIA RTX 2070 SUPER GPU and VRAM.

## Operational Observations

- Intermittent QUIC connectivity failures were observed in `cloudflared` logs; the deployment recovered automatically. Track this primarily as an availability and monitoring observation unless further evidence changes the assessment.
- `cloudflared` version `2026.8.2` was observed while its logs recommended `2026.8.3`.
- The deployment uses the floating image tag `cloudflare/cloudflared:latest`. Pinning a controlled version is a supply-chain and change-control remediation candidate.
