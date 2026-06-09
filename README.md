# homelab

> Unified self-hosted apps for `catcatz.cc` — single Cloudflare Tunnel + Caddy reverse proxy. Clean subfolder routing (`/tutor/`, `/trip/`, etc.).

**Status:** ✅ Production

## Architecture

```
Internet → Cloudflare Tunnel → Caddy (:80) → Apps
                (catcatz.cc)          /tutor/ → ai-english-tutor (8001)
                                      /trip/  → nagoya-travel (static)
```

## What's Inside

- `caddy/Caddyfile` — Main Caddy configuration (subfolder routing)
- `cloudflared/cloudflared.service` — Systemd service for Cloudflare Tunnel
- `SPEC.md` — Original planning document

## Current Apps

| Path | App | Port | Type |
|------|-----|------|------|
| `/tutor/` | ai-english-tutor | 8001 | FastAPI backend |
| `/trip/` | nagoya-travel | static | HTML/JS frontend |

## Quick Commands

```bash
# Restart Caddy
sudo systemctl restart caddy

# Restart Cloudflare Tunnel
sudo systemctl restart cloudflared

# View logs
sudo journalctl -u caddy -f
sudo journalctl -u cloudflared -f
```

## DNS / Tunnel

- Tunnel name: `homelab`
- Public hostname: `catcatz.cc` → `http://localhost:80`
- All subfolder routing handled by Caddy

## Notes

- Switched from Traefik to Caddy for simpler subfolder routing
- No Docker needed for current apps (Caddy serves static + reverse proxy)
- Future apps can be added easily under new paths in Caddyfile
