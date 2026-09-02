# Security Findings

This document tracks security observations discovered during the Home Lab Security & Operations project.

Finding IDs are retained even after remediation so that changes remain historically traceable.

---

## SEC-001 - WireGuard LXC Permissive Host Firewall

**Status:** Open  
**Severity:** Review Required  
**Discovered:** 2026-08-23

The WireGuard LXC currently uses permissive firewall policies.

Observed IPv4 policies:

- INPUT ACCEPT
- FORWARD ACCEPT
- OUTPUT ACCEPT

Observed IPv6 policies:

- INPUT ACCEPT
- FORWARD ACCEPT
- OUTPUT ACCEPT

WireGuard intentionally listens on UDP 51820.

SSH also listens on TCP 22 on all interfaces.

### Risk

The Xfinity gateway currently protects the host from unsolicited IPv4 connections except for UDP 51820.

However, the WireGuard LXC also has a globally routable IPv6 address. If unsolicited IPv6 connections are permitted through the Xfinity firewall, SSH or other future listeners may be directly reachable from the Internet.

### Next Steps

- Verify external IPv6 reachability.
- Review Xfinity IPv6 firewall behavior.
- Design host firewall policy.
- Restrict management services to trusted networks or VPN where appropriate.

No firewall changes have been made yet.

---

## SEC-002 - Inconsistent Proxmox Guest Firewall Configuration

**Status:** Open  
**Severity:** Informational / Review Required  
**Discovered:** 2026-08-23

Proxmox VM NIC firewall settings are inconsistent.

Examples:

- FooballPoolDB01: firewall enabled
- k3s-cp-01: firewall disabled
- k3s-cp-02: firewall disabled
- k3s-cp-03: firewall disabled
- k3s-wk-01: firewall disabled
- k3s-wk-02: firewall disabled
- k3s-wk-03: firewall flag not explicitly configured
- k3s-lb-01: firewall flag not explicitly configured

### Risk

There is currently no consistent Proxmox guest firewall strategy.

### Next Steps

Determine the intended security-control boundaries across:

- network VLAN/firewall rules
- Proxmox firewall
- guest OS firewalls
- Kubernetes NetworkPolicies

No firewall settings will be changed until the network segmentation design is complete.

---

## SEC-003 - Globally Routable IPv6 on Infrastructure

**Status:** Open  
**Severity:** Review Required  
**Discovered:** 2026-08-23

Multiple infrastructure systems receive globally routable IPv6 addresses.

Verified systems include:

- WireGuard
- K3s load balancer
- K3s control-plane nodes
- K3s worker nodes

Observed IPv6 prefix:

`2601:281:c900:c220::/64`

### Risk

IPv6 does not rely on IPv4 NAT port forwarding.

The Xfinity gateway IPv6 firewall behavior therefore becomes part of the Internet attack surface.

Globally scoped IPv6 addressing alone does not prove that these systems are reachable from the Internet.

### Next Steps

- Perform external IPv6 reachability testing.
- Determine Xfinity inbound IPv6 firewall policy.
- Inventory IPv6 listeners.
- Decide whether IPv6 is required for the home lab.
- Implement explicit IPv6 firewall controls if required.

---

## SEC-004 - Legacy Kubernetes NodePort

**Status:** Resolved  
**Severity:** Low / Review Required  
**Discovered:** 2026-08-23
**Resolved:** 2026-08-23

The Kubernetes service `default/nginx-test` exposes:

Workload was obsolete and removed

`TCP 31733`

using a NodePort service.

The service has existed for approximately 133 days.

### Risk

NodePort services listen through Kubernetes nodes and can unintentionally increase the internal or external attack surface.

### Next Steps

- Determine whether nginx-test is still required.
- Identify the backing deployment/pods.
- Verify reachability.
- Remove the service if it is obsolete.

---

## SEC-005 - Insufficient Kubernetes East-West Segmentation

**Status:** Open / In Progress
**Severity:** Medium

K3s uses Flannel for the pod data plane and active kube-router NetworkPolicy enforcement. Policy coverage now includes ArgoCD, Immich Redis, and the successfully segmented Celestial and Travel Planner pilots. Most namespaces still have no NetworkPolicy and remain default-allow for east-west traffic.

Verified policy observations include:

- Immich Redis permits ingress on TCP `6379` without a source selector and has unrestricted egress.
- ArgoCD Redis is more tightly restricted to specific ArgoCD components.
- Several ArgoCD policies use `namespaceSelector: {}`, allowing ingress from any namespace.
- `argocd-server` has `ingress: - {}`, effectively permitting unrestricted ingress to the selected pod.
- Internet-facing workloads, including `cloudflared` and the applications it publishes, share the cluster with unrelated workloads and sensitive management services without meaningful namespace-level isolation.

### Risk

A compromised Internet-facing workload could use the mostly default-allow east-west network posture for lateral movement toward unrelated application, data, or management workloads. Active NetworkPolicy enforcement provides a control mechanism, but current policy coverage does not establish broad isolation.

### Pilot Progress - Celestial

Celestial is the first successfully segmented pilot workload. Its four policies establish default-deny ingress and egress with explicit allowances for the Cloudflare Tunnel production path, CoreDNS over TCP/UDP `53`, PostgreSQL at `10.0.0.129:5432`, and external HTTPS APIs over TCP `443`. The generic Internet HTTPS rule excludes private RFC1918 destinations unless explicitly permitted.

The policies passed Kubernetes server-side dry run, synchronized automatically through ArgoCD, and were confirmed active in the `celestial` namespace. The public health endpoint remained healthy over the production Cloudflare Tunnel path, while a temporary pod in the unrelated `default` namespace resolved the Celestial service but could not connect directly. This demonstrates that kube-router NetworkPolicy enforcement works without breaking Celestial production traffic and reduces lateral-movement opportunities from that workload.

The unused Traefik allowance for `celestial.lab.local` is a cleanup candidate. After confirming the LAN hostname is unnecessary, remove that allowance and consider removing the unused Celestial Ingress resource as a separate change.

### Pilot Progress - Travel Planner

Travel Planner is the second successfully segmented pilot. The `travel-planner-api` workload in namespace `travel-planner` now has default-deny ingress and egress with explicit allowances for `cloudflared` ingress to TCP `8000`, CoreDNS, PostgreSQL at `10.0.0.129:5432`, and public HTTPS. The HTTPS rule excludes RFC1918 destinations. Traefik is also currently allowed to TCP `8000`, but `travel.lab.local` is not an active production frontend path and that allowance is a possible cleanup item.

The four policies passed server-side dry run and were deployed through ArgoCD. The public health endpoint returned HTTP/2 `200` after the correct `cloudflared` allowance was committed, while an arbitrary pod in `default` could resolve the service but could not establish a TCP connection. This preserves DNS while blocking unauthorized east-west application access.

The pilot also verified GitOps reconciliation. A manual `kubectl` correction was reverted to the older Git state by ArgoCD. Commit `047e80f` (`fix travel planner cloudflare ingress`) then recorded the correction in Git, after which ArgoCD reconciled it and restored the public endpoint. Persistent changes must be made in Git, not only in the cluster.

Before enforcing default-deny ingress, identify the real production ingress path. Travel Planner is served from Vercel through Cloudflare Tunnel, not through the unused `travel.lab.local` Traefik route. Omitting the `cloudflared` dependency initially caused HTTP `502` until the policy was corrected.

### Pilot Results

1. Celestial - successful.
2. Travel Planner - successful.
3. Cluster-wide rollout - still in progress.

### Design Constraints

Future segmentation must preserve known required traffic:

- Open WebUI in namespace `ai` to Ollama at `10.0.0.41:11434`
- Kubernetes workloads to PostgreSQL dependencies at `10.0.0.129`
- `cloudflared` in namespace `immich` to published application services across namespaces
- Prometheus cross-namespace scraping
- DNS access to CoreDNS in `kube-system`
- Media and NFS traffic, Harbor, Longhorn, and other verified cross-namespace service dependencies

### Next Steps

- Continue one application or namespace at a time using discovery, dependency mapping, default-deny, explicit allow, and validation.
- Avoid cluster-wide blanket default-deny policies that could disrupt DNS, Cloudflare Tunnel, Traefik, Prometheus, external PostgreSQL, media/NFS traffic, Harbor, Longhorn, or other cross-namespace services.
- Remove the unused Celestial Traefik allowance after confirming `celestial.lab.local` is unnecessary.
- Review the Travel Planner Traefik allowance and remove it after confirming `travel.lab.local` remains unnecessary.

The overall finding remains open because most namespaces are still unsegmented. The successful workload pilots do not establish cluster-wide east-west isolation.

## SEC-006 — Review Internet exposure controls for Hermes

**Status:** Open  
**Severity:** Review Required

Hermes/Open WebUI is verified as Internet-accessible through Cloudflare Tunnel:

`hermes.barnesfamily-pics.online -> open-webui.ai.svc.cluster.local:8080`

The previously documented statement that Hermes is protected by a specific Cloudflare Access policy is not retained as verified current state. The effective Access policy and identity restrictions require validation.

### Planned Review

- Decide whether Hermes should remain publicly accessible, be protected by Cloudflare Access with explicitly authorized identities, or become VPN-only.
- Validate the effective Cloudflare Access policy before treating it as a compensating control.

This is particularly important before Hermes receives any privileged automation capability.

---

## SEC-007 — Ollama LAN listener scope

**Status:** Open
**Severity:** Security Hardening Observation

The Ollama backend on the Windows home PC was observed listening on `0.0.0.0:11434` at LAN address `10.0.0.41`. Open WebUI requires access to this API across the LAN, but the wildcard bind may permit access from other reachable systems.

### Planned Review

- Identify every system that requires the Ollama API.
- During future segmentation and firewall design, restrict access to required consumers, particularly Open WebUI.
- Do not classify the listener as Internet-accessible without external reachability evidence.

---

## SEC-008 — Cloudflare tunnel availability and image control

**Status:** Open
**Severity:** Operational / Supply-Chain Hardening Observation

The `cloudflared` deployment recovered automatically from intermittent QUIC connectivity failures. This is currently an availability and monitoring observation, not a confirmed security vulnerability.

The following version-management observations were also verified:

- Running version observed: `2026.8.2`
- Version recommended by logs: `2026.8.3`
- Deployment image reference: `cloudflare/cloudflared:latest`

### Planned Review

- Monitor and alert on sustained tunnel connectivity failures.
- Evaluate the recommended upgrade through the normal change process.
- Pin `cloudflared` to a controlled version instead of a floating tag.

---

## Network Segmentation Scope

Treat ArgoCD, Longhorn, Vault, Harbor, Prometheus, Grafana, and Alertmanager as management-plane services in future segmentation design. This classification is planning guidance; no network or firewall changes were made as part of this assessment.

---

## Additional Hardening Observations

The Cloudflare deployment currently:

- uses floating image tag `cloudflare/cloudflared:latest`
- has no configured CPU/memory resource requests or limits

These are operational hardening opportunities rather than confirmed vulnerabilities.

---

## Change Policy

No remediation should be performed solely because an item appears in this register.

Workflow:

1. Discover
2. Verify
3. Document
4. Assess impact
5. Design remediation
6. Implement
7. Validate
8. Update documentation

