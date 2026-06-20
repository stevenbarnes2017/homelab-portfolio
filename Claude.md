# CLAUDE.md — Homelab & Projects: Current State + Roadmap

> **Last updated:** 2026-06-10
> Living document. Update after each working session — add, check off, or remove items as priorities shift.

---

## 🧭 Quick Orientation

| Project | Status | Next Action |
|---|---|---|
| Sunday Pickems | Live in prod, pre-season prep | Past-week guard + mock season test |
| Celestial Engine | Migrated to homelab, cleanup pending | Cancel Render, CI/CD |
| Trail Planner | Backend still on Render | Migrate to k3s (Celestial pattern) |
| Homelab | Stable, growth projects queued | App-of-apps review, music server |
| Portfolio | In progress | Credential scrub → public repo |

---

## 🏗️ Infrastructure Snapshot

**Proxmox hosts**
- `lab2` — 10.0.0.232 (ThinkStation P320, 32GB RAM, 4 empty slots)
- `steven` — 10.0.0.237 (ThinkCentre M900)

**k3s cluster (6 nodes)**
- Control planes: 10.0.0.120–122
- Workers: `wk-01/02/03` at 10.0.0.123–125
- Load balancer (MetalLB): 10.0.0.119
- Management: `ansible-control` at 10.0.0.126, manifests at `/opt/k3s-infrastructure/kubernetes/`

**Platform services**
- GitOps: ArgoCD → `stevenbarnes2017/k3s-proxmox-gitops` (app-of-apps pattern)
- Registry: Harbor at `hub.lab.local` (image scanning active)
- Storage: Longhorn | Ingress: Traefik + cert-manager | Public access: Cloudflare tunnels
- Database VM: `footballpooldb01` at 10.0.0.129 (PostgreSQL)
- DNS: Pi-hole at 10.0.0.50 (LXC 150) | VPN: WireGuard LXC 113 (`stevenslab.duckdns.org`)
- Secrets: HashiCorp Vault + Vaultwarden (self-hosted, complete)

**Observability**
- kube-prometheus-stack + Grafana + Loki + Alertmanager (critical-only email)
- 4 provisioned dashboards: Cluster Overview, Node Deep Dive, Services Health, Proxmox Hosts

**Backup/DR**
- Velero + MinIO on `steven` (daily, 72h TTL, immich excluded)
- Proxmox jobs staggered: lab2 04:00, steven 04:30 → `backup-local` NFS
- Runbook in `stevenbarnes2017/homelab-portfolio`

**Other deployed apps**
- Immich (900Gi Longhorn PVC on wk-01 dedicated SATA, Google Photos migration complete)
- Vaultwarden (complete)
- Hommar

---

## 🏈 Sunday Pickems (sundaypickems.com)

Flask / SQLAlchemy / PostgreSQL · k3s via ArgoCD · GitHub Actions CI/CD · Cloudflare tunnel

### ⚠️ Critical — before NFL preseason (August)
- [ ] **Past-week guard on `dispatch_due_reminders`** — prevent stale ReminderJob rows flushing on restart (reminder spam incident, 2026-06-09)


### Backlog
- [ ] Week dropdown: limit to current + past weeks only (currently shows future weeks)
- [ ] Twilio SMS completion (~$10/season for ~20 users; partially wired)
- [ ] PWA push notifications (preferred over SMS long-term — may replace Twilio item)

### Recently completed
- ✅ Tiebreaker pick feature (end-of-season, closest-to-total, last game, optional, lockout at kickoff)
- ✅ Unnested `dispatch_due_reminders` + 5-min scheduler job
- ✅ Slug generation/collision handling, open redirect fix, Settings null guards (7 routes)
- ✅ Password security hardening, logging cleanup, duplicate import cleanup
- ✅ DB reset to `2026 PRE week 1`, ready for August

---

## 🔮 Celestial Engine (celestial-api.sundaypickems.com)

Flask/gunicorn · migrated Render → homelab k3s (2026-06-09, saves ~$14/mo) · `celestial` namespace

### Cleanup
- [x] Cancel Render subscription 💸
- [ ] Commit `GROQ_API_KEY` to GitOps deployment manifest
- [ ] GitHub Actions CI/CD

### Known issues
- [ ] Timezone bug: server runs UTC → evening Mountain Time users see next day's date

---

## 🥾 Trail Planner

React/Vite PWA (Vercel) · FastAPI backend (still on Render) · JWT auth complete · saved trips + wishlist live · wife is a user

### Next
- [X] **Migrate FastAPI backend to homelab k3s** — Celestial pattern:
  Dockerfile → build/push Harbor → namespace + secrets → ArgoCD manifests → Cloudflare tunnel route → fix hardcoded frontend API URL
- [X] Dockerfile (doesn't exist yet)

### Future features
- [ ] Road trip mode (multi-stop map + Google Maps waypoints)
- [ ] Email itinerary delivery
- [ ] Affiliate links, custom domain
- [ ] Potential React Native version

---

## 🏠 Homelab Roadmap

### Active / near-term
- [x] ArgoCD app-of-apps review — specific issues unknown, needs investigation
- [ ] 🎵 **Music server stack (added 2026-06-09):** Navidrome + Lidarr + slskd on k3s via ArgoCD
  - First decision: Longhorn vs NFS volume for music library
  - beets for existing collection cleanup
  - Acquisition: Bandcamp / Qobuz / CD rips (replaces Spotify)
- [ ] 🛡️ **Intrusion detection (added 2026-06-09):** Falco DaemonSet + CrowdSec
  - Integrates with kube-prometheus-stack / Alertmanager
  - Defer Suricata/Zeek (noise/complexity); pairs well with VLAN project
- [ ] 🔄 **Ansible inventory unification (added 2026-06-10):**
  - Expand Ansible coverage beyond k3s — all VMs/LXCs (dns01, footballpooldb01, WireGuard, etc.) in one inventory
  - Proxmox dynamic inventory plugin (`community.general.proxmox`) as source of truth — auto-discover VMs/LXCs instead of static hosts.ini
  - Auto-sync Pi-hole local DNS records from inventory (templated pihole.toml hosts block or API) — every host gets a lab.local name automatically
  - Outcome: one inventory file, everything in DNS, nothing managed by hand
### Monitoring / unresolved
- [ ] NFS instability on `steven` (nfsd deadlock under backup load) — soft mounts added, not fully resolved
- [X] Pi-hole DNS config review

### Hardware
- [X] lab2 RAM: ATP 8GB DDR4-2400 ECC RDIMMs → 32GB to 64GB
- [ ] lab2 storage: Samsung SM961 1TB M.2 NVMe PCIe Gen 3

### Learning projects
- [ ] VLANs + cloud VPC (network segmentation)
- [ ] Windows domain controller
- [ ] Fun new k8s apps / LXC services (open-ended)

---

## 💼 Portfolio


- [ ] GitHub documentation polish
- [ ] LinkedIn updates
- [ ] Portfolio site

**Public repos:** `NFL_Football_Pool`, `homelab-portfolio`

---

## 📒 Hard-Won Lessons (don't relearn these)

- **ArgoCD selfHeal + RWO Longhorn = footgun.** Helm charts with dynamic annotations (Harbor) trigger drift → rollout → new pods stuck on RWO volumes. Fix: `ignoreDifferences` on deployment annotations.
- **GitOps discipline:** changes go to Git, never `kubectl apply` direct. Orphaned manifests = ArgoCD endlessly recreating deleted resources.
- **Incremental changes with testing between steps** — batching caused travel app regressions.
- **Verify existing code before writing new code.**
- **Mock/test before live season** — the reminder spam incident only stayed harmless because `EMAIL_TEST_MODE=1`.
- **PromQL gotchas:** `kube_pod_status_phase` needs `== 1`; `kube_pod_status_ready` needs `max()` in stat panels; zero-state panels need `or vector(0)` + `"instant": true`.

---

## 🔧 Workflow Reference

- Cluster ops: SSH to `ansible-control` via MobaXterm
- Dev: Windows + VSCode/PowerShell; football repo at `E:\OneDrive\Git Repos\Football_Retry`
- Email: Brevo | AI: Groq (Llama 3.3 70B) | Maps: Google Places API + Mapbox GL JS
- Docs as portfolio artifacts → runbooks in `homelab-portfolio`
