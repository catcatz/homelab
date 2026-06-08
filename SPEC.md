# 🏠 Homelab — Unified Self-Hosted Apps Pipeline

> A Cloudflare Tunnel + Traefik reverse-proxy stack that unifies all self-hosted apps behind a single `catcatz.cc` domain — no more quick-tunnel re-creates, no more per-app DNS, no more port forwarding.

**Status:** 📋 **Spec — Awaiting Approval**
**Created:** 2026-06-08
**Repo:** https://github.com/catcatz/homelab
**Inspired by:** [fire3san/homelabpipeline](https://github.com/fire3san/homelabpipeline) (simpler, single-host adaptation)

---

## 🎯 The Problem

You currently have **two** different hosting patterns for your self-hosted apps:

| App | Pattern | Pain |
|---|---|---|
| `trip.catcatz.cc` | Named Cloudflare Tunnel (systemd service) | ✅ Stable URL, but tied to one app |
| `ai-english-tutor` | Cloudflare **quick tunnel** (trycloudflare.com) | ❌ URL changes every restart; have to re-share link |

You told me you have **more apps coming**. Solving this once now means each new app takes 5 minutes to deploy, not 30.

---

## 💡 The Solution: One Tunnel + One Reverse Proxy

```
Internet (HTTPS)
    ↓
Cloudflare Edge (wildcard *.catcatz.cc)
    ↓ outbound tunnel
cloudflared (1 named tunnel — handles ALL apps)
    ↓
Traefik (reverse proxy — reads Docker container labels)
    ↓
┌──────────┬───────────────┬──────────┬──────────┐
│  trip    │  tutor        │  app3    │  appN    │
│  :8080   │  :8001        │  :8002   │  :8003   │
└──────────┴───────────────┴──────────┴──────────┘
```

**URLs (after this is set up):**
- `https://trip.catcatz.cc` → trip server (:8080)
- `https://tutor.catcatz.cc` → ai-english-tutor (:8001)
- `https://app3.catcatz.cc` → new app (:8002)
- (add as many as you want)

**Or, if you prefer single-domain subfolders** (option B):
- `https://catcatz.cc/trip/` → trip
- `https://catcatz.cc/tutor/` → ai-english-tutor
- `https://catcatz.cc/app3/` → new app

---

## 📐 Architecture (Detailed)

```
┌────────────────────────────────────────────────────────────┐
│ CLOUDFLARE (Zero Trust)                                    │
│                                                            │
│  - catcatz.cc is already on Cloudflare ✅                  │
│  - Wildcard DNS: *.catcatz.cc → tunnel endpoint            │
│  - Free TLS + DDoS protection                             │
└─────────────────────────┬──────────────────────────────────┘
                          │ outbound 443 (no inbound ports)
┌─────────────────────────▼──────────────────────────────────┐
│ UBUNTU HOST (192.168.1.217) — your existing VM            │
│                                                            │
│  ┌────────────────────────────────────────────────────┐   │
│  │ systemd service: cloudflared                       │   │
│  │   /usr/bin/cloudflared --token <NAMED_TUNNEL>      │   │
│  │   Replaces current trycloudflare quick tunnel      │   │
│  └────────────────────────┬───────────────────────────┘   │
│                           ↓                                │
│  ┌────────────────────────────────────────────────────┐   │
│  │ Docker container: traefik (port 80, 8080)          │   │
│  │   - Listens on :80 (plain HTTP — TLS at CF edge)   │   │
│  │   - Dashboard on :8080 (LAN only)                  │   │
│  │   - Auto-discovers containers via Docker labels    │   │
│  └────────────────────────┬───────────────────────────┘   │
│                           ↓                                │
│  ┌──────────────┬──────────────┬──────────────────────┐    │
│  │ trip         │ tutor        │ future apps          │    │
│  │ (existing    │ (existing    │ (just add compose)   │    │
│  │  :8080)      │  :8001)      │                      │    │
│  └──────────────┴──────────────┴──────────────────────┘    │
│                                                            │
│  ┌────────────────────────────────────────────────────┐   │
│  │ Docker container: portainer (optional, LAN only)   │   │
│  │   - https://<lan-ip>:9443                          │   │
│  │   - Web UI to manage stacks/containers             │   │
│  └────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Stack (What Gets Installed)

| Component | Purpose | Install Method |
|---|---|---|
| **Docker Engine** + Compose | Run all apps as containers | `apt install docker.io` |
| **cloudflared** (named tunnel) | Single outbound tunnel for everything | Already installed ✅ (reconfigure) |
| **Traefik v3** | Reverse proxy, Docker auto-discovery | Docker image |
| **Portainer CE** | (Optional) Web UI for Docker | Docker image |
| **proxy-network** | Shared Docker bridge network | Created by us |

**NOT included** (per your spec — keep it simple):
- ❌ GitHub Actions / CI/CD (later if you want)
- ❌ Prometheus / Grafana monitoring (later)
- ❌ Multiple VMs (we use your existing Ubuntu host)

---

## 📋 Implementation Plan (Step-by-step)

### Phase 0: Cleanup (5 min)
- [ ] Stop current `cloudflared` quick tunnel (kill `proc_1383869b1edc` if still running)
- [ ] Stop systemd `cloudflared` service (we'll reconfigure it)
- [ ] Verify no tunnel-related port conflicts (we'll route via Traefik :80)

### Phase 1: Docker Setup (10 min)
- [ ] `apt install docker.io docker-compose-v2` (needs your approval — **will not auto-install**)
- [ ] Add user to `docker` group: `sudo usermod -aG docker catcatz`
- [ ] Create shared network: `docker network create proxy-network`
- [ ] Verify: `docker run hello-world`

### Phase 2: Create Homelab Repo ✅ DONE
- [x] Init `~/homelab/` with this SPEC
- [x] Create GitHub repo: https://github.com/catcatz/homelab
- [ ] Push initial SPEC

### Phase 3: Traefik Stack (15 min)
- [ ] Write `stacks/traefik/docker-compose.yml` (Traefik + Portainer)
- [ ] Write `stacks/traefik/traefik.yml` (static config)
- [ ] Write `stacks/traefik/dynamic.yml` (middleware for path stripping)
- [ ] `cd stacks/traefik && docker compose up -d`
- [ ] Verify dashboard: `http://192.168.1.217:8080` (LAN)

### Phase 4: Cloudflare Named Tunnel (20 min)
- [ ] Login to https://one.dash.cloudflare.com/
- [ ] Create tunnel `homelab` (replaces existing `trip` tunnel — or keep both if you want separation)
- [ ] Add public hostnames: `*.catcatz.cc` → `http://traefik:80`
- [ ] Update systemd service with new tunnel token
- [ ] Restart systemd `cloudflared`
- [ ] Verify: `curl -H "Host: test.catcatz.cc" http://localhost:80` returns 404 from Traefik (proving routing works)

### Phase 5: Migrate Existing Apps (30 min)

**5a. ai-english-tutor → Docker**
- [ ] Write `stacks/tutor/docker-compose.yml` with Traefik labels:
  ```yaml
  services:
    tutor:
      build: ../../ai-english-tutor
      networks: [proxy-network]
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.tutor.rule=Host(`tutor.catcatz.cc`)"
        - "traefik.http.services.tutor.loadbalancer.server.port=8001"
  ```
- [ ] Stop existing uvicorn (kill 29171)
- [ ] `docker compose up -d`
- [ ] Verify: `curl https://tutor.catcatz.cc/health` returns 200

**5b. trip → Docker** (if you have a Dockerfile / compose already)
- [ ] Add to `stacks/trip/docker-compose.yml` with similar Traefik labels
- [ ] Hostname: `trip.catcatz.cc`, port: 8080
- [ ] Verify: `curl https://trip.catcatz.cc` returns 200

### Phase 6: Documentation (10 min)
- [ ] Write `README.md` (architecture, app onboarding, troubleshooting)
- [ ] Write `docs/ADDING_A_NEW_APP.md` (5-minute recipe)
- [ ] Add to existing project memory: "All apps deployed via homelab stack"

---

## 🆕 Adding a New App (Future — 5 minutes)

```bash
# 1. Create stack folder
mkdir -p ~/homelab/stacks/newapp && cd ~/homelab/stacks/newapp

# 2. Create docker-compose.yml
cat > docker-compose.yml <<'EOF'
services:
  newapp:
    build: ../../newapp-source  # or image: ...
    networks: [proxy-network]
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.newapp.rule=Host(`newapp.catcatz.cc`)"
      - "traefik.http.services.newapp.loadbalancer.server.port=8080"
networks:
  proxy-network:
    external: true
EOF

# 3. Add DNS in Cloudflare (or use wildcard — auto-routes)
# 4. Start it
docker compose up -d
# 5. Done — live at https://newapp.catcatz.cc
```

**For subfolder routing** (alternative): replace the `Host()` rule with `PathPrefix(`/newapp`)` and add strip-prefix middleware.

---

## 🔒 Security Considerations

- **No inbound ports opened** — only outbound 443 to Cloudflare
- **TLS terminated at Cloudflare** — internal traffic is plain HTTP (LAN is trusted)
- **Traefik dashboard** (`:8080`) bound to LAN IP only — never exposed via tunnel
- **Cloudflare Access policies** (optional, future) — restrict by email/SSO
- **Docker socket** is NOT exposed to app containers (security best practice)

---

## ⚠️ Risks & Open Questions

### Q1: Subdomain (default) vs Subfolder routing?

**My recommendation: SUBDOMAIN** (Phase 5 default)
- ✅ Cleaner URLs
- ✅ Cookies / sessions don't collide
- ✅ CORS is straightforward
- ✅ Apps can use absolute paths freely

**Subfolder** (alternative)
- ✅ Single domain certificate
- ❌ Apps with absolute paths break
- ❌ Session cookies shared by default
- ❌ Some apps hardcode `/api/...` and choke

### Q2: Replace existing `trip` tunnel, or keep it?

**Recommendation: REPLACE.** One tunnel is simpler. But if `trip` has special access policies (e.g. email-restricted), we can keep it as-is and add a second tunnel `homelab` for tutor + future apps. **You decide.**

### Q3: Docker install — needs your approval.

You said: "你以後安裝野一定要問左我先". So I will ask before `apt install docker.io`. (I will NOT auto-install.)

### Q4: Will ai-english-tutor need Dockerfile changes?

Probably yes — to add a `Dockerfile` at `~/ai-english-tutor/Dockerfile`. The existing `requirements.txt` + `app/main.py` should work inside a container with minimal changes. I'll write the Dockerfile as part of Phase 5a.

---

## 📦 Repo Structure (Target)

```
homelab/
├── README.md                          ← Quick start, architecture
├── SPEC.md                            ← This file
├── docs/
│   ├── ADDING_A_NEW_APP.md           ← 5-min recipe
│   ├── MIGRATION.md                  ← Move from quick tunnel to named
│   └── TROUBLESHOOTING.md
├── stacks/
│   ├── traefik/
│   │   ├── docker-compose.yml        ← Traefik + Portainer
│   │   ├── traefik.yml               ← Static config
│   │   └── dynamic/
│   │       └── middlewares.yml       ← StripPrefix, etc.
│   ├── tutor/
│   │   └── docker-compose.yml        ← ai-english-tutor
│   ├── trip/
│   │   └── docker-compose.yml        ← trip server
│   └── example-app/                   ← Template for new apps
│       └── docker-compose.yml
└── .gitignore
```

---

## ⏱️ Time Estimate

| Phase | Duration | What you do |
|---|---|---|
| 0. Cleanup | 5 min | None — I do it |
| 1. Docker install | 10 min | **Approve `apt install docker.io`** |
| 2. Repo init | 1 min | None — done ✅ |
| 3. Traefik | 15 min | None |
| 4. Cloudflare tunnel | 20 min | **Login to CF dashboard (10 min for the hostname setup)** |
| 5. Migrate apps | 30 min | None |
| 6. Docs | 10 min | None |
| **Total** | **~90 min** | **You: ~15 min interactive (mostly CF dashboard)** |

---

## 🚦 Approval Gate

Please review and respond with one of:

✅ **"Go"** — I'll execute Phase 0 → 6, asking for approval only at the Docker install + Cloudflare dashboard steps.

🔧 **"Subfolder routing instead"** — I'll adapt Phase 3+ configs for `catcatz.cc/<app>/` instead of `<app>.catcatz.cc`.

❌ **"Keep both tunnels"** — I'll create a new `homelab` tunnel alongside existing `trip` tunnel, no disruption.

❓ **"Tell me more about X"** — I'll explain any section in depth before committing.

---

**End of spec. Waiting for your call. 💪**
