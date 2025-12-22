# Deploying Services

Step-by-step guide for deploying new services on arise-server.

## Decision Tree

```
Can you modify the docker-compose.yml?
├── YES → Use Traefik routing (simpler)
│         Add domain in DokPloy, no port mapping needed
│
└── NO (external repo) → Use direct Cloudflare route
                         Requires unique host port + tunnel route
```

## Method 1: Traefik Routing (Recommended)

For services where you control the docker-compose.yml.

### Step 1: Prepare Compose File

**Single-container** (sandboxes):
```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    networks:
      - dokploy-network
    environment:
      - DB_TYPE=sqlite
    volumes:
      - n8n_data:/home/node/.n8n

networks:
  dokploy-network:
    external: true

volumes:
  n8n_data:
```

**Multi-container** (production with database):
```yaml
services:
  app:
    image: myapp:latest
    networks:
      - dokploy-network
      - internal
    depends_on:
      - postgres

  postgres:
    image: postgres:17-alpine
    networks:
      - internal
    volumes:
      - postgres_data:/var/lib/postgresql/data

networks:
  dokploy-network:
    external: true
  internal:

volumes:
  postgres_data:
```

### Step 2: Create in DokPloy

1. Go to DokPloy → Project → Add Compose
2. Paste your compose file
3. Set environment variables
4. Click Deploy

### Step 3: Add Domain

1. Go to service → Domains → Add Domain
2. Configure:
   - **Host**: `myservice.arisegroup-tools.com`
   - **Port**: Container's internal port (e.g., 5678)
   - **Service Name**: Must match service name in compose (e.g., `n8n`)
   - **HTTPS**: **OFF** (Cloudflare handles this)
3. Save

### Step 4: Redeploy

Domain changes require redeploy to regenerate Traefik labels.

1. Go to service → Click Redeploy
2. Wait for containers to restart

### Step 5: Verify

```bash
curl -I https://myservice.arisegroup-tools.com/
# Should return HTTP 200
```

## Method 2: Direct Cloudflare Route

For external repos or services needing specific port access.

### Step 1: Choose Unique Port

Check existing port usage:
```bash
ssh user@arise-server "docker ps --format '{{.Ports}}'"
```

Pick next available (e.g., 5683).

### Step 2: Create Compose with Port Mapping

```yaml
services:
  app:
    image: myapp:latest
    ports:
      - "5683:8080"  # HOST:CONTAINER
```

### Step 3: Add Tunnel Route

```bash
./run modules/infrastructure/tool/cloudflare_api.py tunnel route-add \
  86c6d6ed-a7e0-48df-a50a-fc0766ff5ab6 \
  myservice.arisegroup-tools.com \
  http://localhost:5683
```

### Step 4: Deploy in DokPloy

Create and deploy the compose (no domain needed in DokPloy).

### Step 5: Verify

```bash
curl -I https://myservice.arisegroup-tools.com/
```

## Deployment Checklist

- [ ] **Check compose ownership** — Can you modify docker-compose.yml?
- [ ] **Add network config** — `dokploy-network` external, dual-network for multi-container
- [ ] **Add domain in DokPloy** (Traefik method):
  - Correct port (container internal, not host)
  - Correct serviceName
  - **HTTPS: OFF**
- [ ] **Redeploy** — Required after domain changes
- [ ] **Verify** — Check URL returns 200, check logs for errors

## Removing Services

**Traefik-routed:**
1. Delete domain in DokPloy
2. Stop/delete compose

**Direct-routed:**
1. Stop/delete compose in DokPloy
2. Remove tunnel route:
   ```bash
   ./run modules/infrastructure/tool/cloudflare_api.py tunnel route-remove \
     86c6d6ed-a7e0-48df-a50a-fc0766ff5ab6 \
     myservice.arisegroup-tools.com
   ```

## Related Docs

- [Networking](networking.md) - Dual-network pattern explained
- [Troubleshooting](troubleshooting.md) - Common deployment issues
