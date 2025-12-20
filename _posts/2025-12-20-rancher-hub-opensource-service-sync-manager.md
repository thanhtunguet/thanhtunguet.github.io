---
title: "Rancher Hub: Open-Source Service Sync Manager for Rancher Clusters"
date: 2025-12-20 10:00:00 +0700
categories: ["DevOps", "Kubernetes"]
tags: ["DevOps", "Kubernetes", "Rancher", "RancherHub", "OpenSource"]
pin: false
mermaid: false
---

### Rancher Hub: Open-Source Service Sync Manager for Rancher Clusters

Managing multiple Kubernetes environments (Dev/Staging/Prod) is hard—especially when each environment lives in a different Rancher cluster/namespace and you need to keep **services** and **ConfigMaps** consistent.

**Rancher Hub** is an open-source **Service Sync Manager for Rancher Clusters** that helps you **compare** and **synchronize** services across environments, with an audit trail and built-in security features.

- Home: `https://thanhtunguet.info/RancherHub/`
- Getting Started: `https://thanhtunguet.info/RancherHub/getting-started`
- Features: `https://thanhtunguet.info/RancherHub/features`

---

### Why Rancher Hub?

If you’ve ever experienced “it works in staging but not in prod”, the root cause is often drift:

- **Different image tags** (or different registries)
- **Different env/config values** (ConfigMaps/variables)
- **Different replicas/settings** between environments

Rancher Hub focuses on making drift **visible** and giving you safe workflows to **sync** the parts you intend to sync.

---

### What you can do with Rancher Hub (highlights)

Based on the official feature documentation, Rancher Hub provides:

#### Authentication & user management

- **JWT-based authentication**
- **Mandatory Two-Factor Authentication (2FA)** (TOTP + QR setup)
- **Trusted devices** (optional, up to 30 days; max 3 devices per user)
- Basic user lifecycle & protected routes

#### Multi-site integration (Rancher + Harbor)

- **Connect to unlimited Rancher instances**
- **Encrypted storage of Rancher API tokens**
- Connection testing/validation
- **Harbor registry integration**: browse repos/tags, track deployments, see storage utilization insights

#### Environment & instance management

- Create logical environments (Dev/Staging/Prod)
- Map each environment to a **cluster + namespace** (app instance mapping)

#### Service management & synchronization

- Auto-discover services from Rancher clusters
- Real-time status + replica info
- **One-click sync** and **batch sync** between environments
- **Audit trail** with user attribution

#### ConfigMap comparison & sync

- Side-by-side ConfigMap comparison (key-by-key with status indicators)
- Sync individual keys or batch sync
- Preview changes + copy-to-clipboard support

#### Monitoring & alerting

- Scheduled health checks and real-time service status monitoring
- **Telegram notifications** (including SOCKS5 proxy support)

#### Audit/history & API automation

- Detailed sync history for services and ConfigMaps (timestamps + source/target)
- REST API with Swagger docs + webhook support

---

### Getting started (from the docs)

The official “Getting Started” guide lists two supported ways to run Rancher Hub.

#### Option 1: Local development (recommended)

Prerequisites:

- Node.js >= 18.0.0
- npm >= 8.0.0
- Docker (optional)
- Access to one or more Rancher clusters with API tokens

Run:

```bash
git clone https://github.com/thanhtunguet/RancherHub.git
cd RancherHub

npm install

cp apps/backend/.env.example apps/backend/.env
cp apps/frontend/.env.example apps/frontend/.env

npm run dev
```

Endpoints:

- Frontend: `http://localhost:5173`
- Backend API: `http://localhost:3000`
- API Docs: `http://localhost:3000/api/docs`

#### Option 2: Docker Compose (production images)

The docs include a Docker Compose setup using:

- `postgres:15-alpine`
- `thanhtunguet/rancher-hub-backend:latest`
- `thanhtunguet/rancher-hub-frontend:latest`
- `nginx:stable-alpine` (reverse proxy on port 8080)

Start services:

```bash
docker-compose up -d
```

Access:

- Frontend: `http://localhost:8080`
- Backend API: `http://localhost:8080/api`
- API Docs: `http://localhost:8080/api/docs`

---

### First login & security notes

On first run, a default admin user is automatically created (per docs):

- Username: `admin`
- Password: `admin123`

You’ll be required to set up **2FA** at first login, and you should immediately:

- Change the default password
- Replace `ENCRYPTION_KEY` and `JWT_SECRET` with secure values (especially in production)

---

### Suggested workflow (Dev → Staging → Prod)

One practical way to adopt Rancher Hub is:

1. **Connect Rancher Sites** (each Rancher instance you manage)
2. Create **Environments** (Dev/Staging/Prod)
3. Create **App Instances** (map environment → cluster/namespace)
4. Use **Service** + **ConfigMap** comparison to spot drift
5. Sync changes intentionally with the audit trail on

---

### Learn more

- Docs home: `https://thanhtunguet.info/RancherHub/`
- Quick start: `https://thanhtunguet.info/RancherHub/getting-started`
- Feature reference: `https://thanhtunguet.info/RancherHub/features`
- Source code: `https://github.com/thanhtunguet/RancherHub`


