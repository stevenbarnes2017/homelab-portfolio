@'
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
'@ | Set-Content -Encoding utf8 .\docs\security\findings.md

## SEC-005 — Cloudflare tunnel pods are not network-isolated.
cloudflared intentionally provides Internet-origin access to six applications, but the pods are not currently restricted to only those destination services. A compromise of a tunnel pod could therefore provide broader east/west access inside the cluster.

The Cloudflare tunnel pods do not appear over-privileged in Kubernetes RBAC, but they are not network-isolated and can likely initiate connections broadly within the cluster.

## SEC-006 — Hermes Cloudflare Access policy is authentication-method based rather than identity-restricted.

**Status:** Open  
**Priority:** Security Hardening

Hermes is protected by Cloudflare Access.

Current Allow policy selects the Cloudflare login method rather than explicitly authorized user identities.

### Planned Review

Restrict Hermes Access to specifically approved identities.

This is particularly important before Hermes receives any privileged automation capability.

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