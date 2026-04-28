# Service: Sunday Pickem's Platform

## Overview

Sunday Pickem's is a Flask-based, multi-tenant web platform that allows users to participate in weekly NFL pick’em pools using confidence-based scoring.

The system is designed to simulate a production application with separate web and background processing components, integrated observability, and Kubernetes-based deployment.
---

## 🧱 Architecture

### Components

- **Web Application**
  - Flask + Gunicorn
  - Handles user interaction, authentication, picks, and dashboards

- **Scheduler Service**
  - Separate Kubernetes deployment
  - Runs APScheduler for background jobs

- **Database**
  - PostgreSQL
  - Local (homelab)
  - Production: Render

---

### Deployment Model

- Dockerized application
- Deployed to Kubernetes (k3s)
- Two deployments:
  - `football-web`
  - `football-scheduler`

- Exposed via Traefik ingress:
  - `football.lab.local`

---

## 🔁 Background Processing (Scheduler)

The scheduler runs as an independent service to decouple background tasks from the web application.

### Responsibilities

- Import weekly schedules
- Fetch betting spreads (external API)
- Send reminder emails
- Finalize weekly results
- Trigger scoring and leaderboard updates

### Design Decision

Separating the scheduler:
- Prevents blocking web requests
- Improves reliability
- Aligns with microservice-style architecture

---

## 👥 Multi-Tenancy Model

The application supports multiple independent groups (pools).

### Core Models

- **PoolGroup**
  - Represents a group/pool
  - Contains users and picks

- **GroupMember**
  - Maps users to groups
  - Role-based access:
    - `global_admin`
    - `group_admin`
    - `user`

---

### Key Features

- Users can belong to multiple groups
- Admin dashboards scoped per group
- Data isolation enforced at query level
- Active group context maintained in session

---

## 📊 Observability

### Metrics

- Exposed via `/metrics` endpoint
- Scraped by Prometheus

#### Key Metrics

- HTTP request metrics (success/error rates)
- Custom metric:
  - `football_current_nfl_week`

---

### Logging

- Logs collected via Promtail
- Stored in Loki
- Viewable in Grafana

---

### Alerting

- Alertmanager configured for email alerts
- Alerts based on Prometheus metrics
- Thresholds currently being tuned

---

## 🔒 Security & Secrets

### Vault Integration

- Secrets stored in HashiCorp Vault
- Injected into Kubernetes pods as environment variables

#### Examples

- Database credentials
- Email service credentials
- API keys

### Benefits

- No secrets stored in code or Git
- Centralized secret management
- Production-aligned security model

---

## 🔐 TLS & Certificates

- Managed via cert-manager
- Self-signed ClusterIssuer for internal services
- Used for securing internal ingress endpoints

---

## ⚙️ CI/CD Pipeline

### Flow

1. Code pushed to GitHub
2. GitHub Actions builds Docker image
3. Image pushed to Harbor registry
4. Kubernetes deployment updated
5. Pods restart with new image

---

## 🧪 Key Features

- Weekly pick submission with confidence scoring
- Multi-group support
- Admin dashboards
- Automated scheduling system
- Live game data ingestion (ESPN API)
- Metrics endpoint for observability

---

## ⚠️ Known Issues / Gaps

- Scheduler logging visibility inconsistent
- No formal SLIs/SLOs defined
- Limited retry/error handling in scheduled jobs
- No rate limiting on external API calls

---

## 🚀 Future Improvements

### Short Term

- SMS notifications (Twilio)
- Improve scheduler logging and visibility
- Add health checks for scheduler

---

### Medium Term

- Define SLIs/SLOs
- Add retry/backoff logic for API calls
- Improve error handling

---

### Long Term

- Scale testing
- Caching layer (Redis)
- External uptime monitoring

---

## 🧠 SRE Considerations

This service demonstrates:

- Service decomposition (web + worker pattern)
- Observability integration (metrics, logs, alerts)
- Secure secret injection (Vault)
- Kubernetes-native deployment model
- CI/CD pipeline automation

---


