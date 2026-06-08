# homelab

> Unified self-hosted apps pipeline for `catcatz.cc` — single Cloudflare Tunnel + Traefik reverse proxy + Docker. No more re-creating quick tunnels, no more per-app DNS, no more port forwarding.

**Status:** 📋 Spec phase. See [SPEC.md](./SPEC.md) for full plan and approval gate.

## TL;DR

This repo will hold the Docker Compose stacks, Traefik config, and documentation for hosting all your self-hosted apps behind one stable `*.catcatz.cc` domain.

```
Internet → Cloudflare (wildcard *.catcatz.cc) → cloudflared → Traefik → Docker apps
```

## What's Inside (Target)

- `stacks/traefik/` — Traefik v3 + Portainer (control plane)
- `stacks/tutor/` — ai-english-tutor
- `stacks/trip/` — trip server
- `stacks/example-app/` — template for new apps
- `docs/` — onboarding, migration, troubleshooting

## Current Status

- [x] Repo created: https://github.com/catcatz/homelab
- [x] `SPEC.md` written, awaiting approval
- [ ] Docker installed on host
- [ ] Traefik stack deployed
- [ ] Cloudflare named tunnel configured
- [ ] ai-english-tutor migrated to Docker
- [ ] trip server migrated to Docker
- [ ] Documentation complete

## Quick Reference

- **Spec:** [SPEC.md](./SPEC.md)
- **Inspiration:** [fire3san/homelabpipeline](https://github.com/fire3san/homelabpipeline) (we're using a simpler single-host variant)
- **Cloudflare dashboard:** https://one.dash.cloudflare.com/
- **Domain:** `catcatz.cc` (already on Cloudflare)
