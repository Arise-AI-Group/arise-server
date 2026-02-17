# Windmill Deployment - 2026-02-17

## What was deployed

Windmill workflow engine at https://windmill.arisegroup-tools.com

**Services:** PostgreSQL 16, Windmill server, default worker, native worker
**Routing:** Traefik → windmill-server:8000
**DokPloy ID:** `km2hC-IELWxzhXoKVEJW_` (shared-services project)
**Compose:** `compose/windmill.yaml`

## Decisions

- **No Caddy/LSP initially** — Windmill's LSP needs WebSocket path routing (`/ws/*` → port 3001) which requires an internal reverse proxy (Caddy). Skipped for simplicity; can add later if code editor intellisense is wanted.
- **Direct Traefik routing** to windmill-server instead of through Caddy.
- **1 default worker + 1 native worker (4 threads)** — conservative for server resources. Scale workers by adding replicas in compose.

## Gotcha: URL-unsafe passwords

First deployment failed silently (DokPloy status: "error", no visible logs). Root cause: `openssl rand -base64 32` generated a password containing `/`, which broke the `DATABASE_URL` postgres connection string interpolation.

**Fix:** Use `openssl rand -hex 24` for any password that gets interpolated into a URL. Updated the cheatsheet with this.

## Post-deployment TODO

- [ ] Change default admin password (`admin@windmill.dev` / `changeme`)
- [ ] Set base URL in Instance Settings to `https://windmill.arisegroup-tools.com`
