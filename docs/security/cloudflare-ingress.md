@'
# Cloudflare Internet Ingress

**Last verified:** 2026-08-23

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