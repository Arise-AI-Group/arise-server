# DokPloy Deployment Cheatsheet

One-page reference for deploying services on arise-server.

## Quick Decision

```
Can you modify the docker-compose.yml?
├── YES → Use Traefik routing (this guide)
└── NO  → Use direct Cloudflare route (ask for help)
```

## Compose Templates

### Single-Container (sandboxes, simple apps)

```yaml
services:
  myapp:
    image: myimage:latest
    restart: unless-stopped
    networks:
      - dokploy-network
    environment:
      - MY_VAR=${MY_VAR}
    volumes:
      - app_data:/data

networks:
  dokploy-network:
    external: true

volumes:
  app_data:
```

### Multi-Container (with database/cache)

```yaml
services:
  myapp:
    image: myimage:latest
    restart: unless-stopped
    networks:
      - dokploy-network  # Traefik reaches this
      - internal         # Talks to database
    depends_on:
      - postgres

  postgres:
    image: postgres:17-alpine
    restart: unless-stopped
    networks:
      - internal         # NOT on dokploy-network (isolated)
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASS}
    volumes:
      - postgres_data:/var/lib/postgresql/data

networks:
  dokploy-network:
    external: true
  internal:

volumes:
  postgres_data:
```

**Key points:**
- Public-facing service: both `dokploy-network` + `internal`
- Internal services (db, cache): `internal` only
- No `ports:` mapping needed for Traefik routing

## Domain Configuration

When adding domain in DokPloy:

| Setting | Value |
|---------|-------|
| Host | `myservice.arisegroup-tools.com` |
| Port | Container's internal port (e.g., 5678, 8080, 3000) |
| Service Name | Exact name from compose file (e.g., `myapp`) |
| **HTTPS** | **OFF** (Cloudflare handles this) |

**After adding domain: REDEPLOY the service.**

## Checklist

- [ ] Compose has `dokploy-network` with `external: true`
- [ ] Multi-container: public service on both networks, internal services on `internal` only
- [ ] Domain added with correct serviceName and port
- [ ] **HTTPS is OFF**
- [ ] Redeployed after adding domain
- [ ] Verified URL returns 200

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| 404 | Domain not added or needs redeploy | Add domain, then redeploy |
| 502 | Can't reach database (`getaddrinfo EAI_AGAIN`) | Use dual-network pattern above |
| 301 loop | HTTPS enabled in DokPloy | Set HTTPS to OFF |
| 504 | Traefik not on dokploy-network | SSH and run: `docker network connect dokploy-network dokploy-traefik` |

## Debugging

```bash
# Check container status
ssh user@arise-server "docker ps | grep myapp"

# View logs
ssh user@arise-server "docker logs -f <container-name>"

# Check networks
ssh user@arise-server "docker inspect <container> --format '{{json .NetworkSettings.Networks}}' | jq"
```
