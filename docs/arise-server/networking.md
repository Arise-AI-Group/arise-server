# Networking

How traffic flows from the internet to containers on arise-server.

## Traffic Flow

```
User Browser
    │
    ▼ HTTPS (TLS termination here)
Cloudflare Edge
    │
    ▼ Tunnel (encrypted)
arise-server
    │
    ▼ HTTP (internal only)
Traefik (routes by hostname)
    │
    ▼ HTTP
Container
```

## Cloudflare Configuration

### DNS
- **Wildcard record**: `*.arisegroup-tools.com` → tunnel CNAME
- No need to add DNS records for new services

### Tunnel
- **Tunnel ID**: `86c6d6ed-a7e0-48df-a50a-fc0766ff5ab6`
- Runs via cloudflared container in DokPloy

### Ingress Rules
```
dokploy.arisegroup-tools.com → localhost:3000 (direct)
n8n.arisegroup-tools.com → localhost:5678 (direct, legacy)
*.arisegroup-tools.com → localhost:80 (Traefik wildcard)
(catch-all) → http_status:404
```

**Important**: Specific routes must come BEFORE the wildcard route.

## Docker Networks

### The Problem

When you add a domain to a DokPloy compose service:
1. DokPloy moves the **main service** to `dokploy-network` (so Traefik can reach it)
2. **Dependent services** (postgres, redis) stay on the compose-default network
3. Main service can't reach dependencies (different networks)

**Symptom**: `getaddrinfo EAI_AGAIN postgres` in logs

### The Solution: Dual-Network Pattern

Containers can be on **multiple networks**:

```yaml
services:
  # Public-facing (Traefik routes here)
  app:
    networks:
      - dokploy-network  # Traefik can reach it
      - internal         # Can talk to database

  # Internal services (isolated)
  postgres:
    networks:
      - internal         # NOT on dokploy-network

networks:
  dokploy-network:
    external: true       # Shared across all projects
  internal:
    # Private to this compose
```

### How It Works

```
┌─────────────────────────────────────────────────┐
│                 dokploy-network                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐         │
│  │ Traefik │  │ proj-A  │  │ proj-B  │         │
│  │         │  │  app    │  │  app    │         │
│  └─────────┘  └────┬────┘  └────┬────┘         │
└────────────────────┼────────────┼──────────────┘
                     │            │
         ┌───────────┴───┐  ┌─────┴───────────┐
         │ proj-A        │  │ proj-B          │
         │ internal      │  │ internal        │
         │  ┌──────────┐ │  │  ┌──────────┐  │
         │  │ postgres │ │  │  │ postgres │  │
         │  └──────────┘ │  │  └──────────┘  │
         └───────────────┘  └────────────────┘
              ISOLATED            ISOLATED
```

- Traefik routes to apps via `dokploy-network`
- Apps talk to their databases via `internal`
- proj-A cannot reach proj-B's postgres

## Traefik Configuration

Traefik is managed by DokPloy. Key points:

- Routes are generated from Docker labels when containers start
- Adding a domain in DokPloy UI requires **redeploy** to take effect
- Must be connected to `dokploy-network` to reach containers

### HTTPS Configuration

**Critical**: Set `https: false` for all domains in DokPloy.

Why:
- Cloudflare handles HTTPS to users
- Cloudflare sends HTTP to Traefik internally
- If Traefik also tries HTTPS redirect → infinite loop

## Related Docs

- [Deploying Services](deploying-services.md) - Step-by-step with network config
- [Troubleshooting](troubleshooting.md) - Network-related issues
