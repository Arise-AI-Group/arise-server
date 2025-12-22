# Troubleshooting

Common issues and how to fix them.

## 404 Not Found

### Domain not added
**Symptom**: URL returns 404 immediately after deployment.

**Cause**: No domain configured for the service.

**Fix**: Add domain in DokPloy → service → Domains.

### Domain added but not redeployed
**Symptom**: Domain was added in DokPloy UI but URL returns 404.

**Cause**: Traefik routes are generated from Docker labels when containers start. The domain config needs a redeploy.

**Fix**: Redeploy the compose service in DokPloy.

### Wrong serviceName
**Symptom**: 404 even after adding domain and redeploying.

**Cause**: The `serviceName` in domain config doesn't match the service name in docker-compose.yml.

**Fix**: Check your compose file for the exact service name (case-sensitive).

## 502 Bad Gateway

### Network isolation
**Symptom**: Main service can't reach database, logs show `getaddrinfo EAI_AGAIN postgres`.

**Cause**: When DokPloy adds a domain, it moves the main service to `dokploy-network`, but dependencies stay on the compose-default network.

**Fix**: Use the dual-network pattern in your compose:
```yaml
services:
  app:
    networks:
      - dokploy-network
      - internal
  postgres:
    networks:
      - internal

networks:
  dokploy-network:
    external: true
  internal:
```

### Container not running
**Symptom**: 502 for a service that was working.

**Cause**: Container crashed or stopped.

**Fix**: Check DokPloy → service → Logs. Restart or redeploy.

## 301/308 Redirect Loop

### HTTPS enabled in DokPloy
**Symptom**: Browser shows "too many redirects" error.

**Cause**: Domain configured with `https: true` in DokPloy. This adds redirect-to-https middleware, but Cloudflare already sends HTTP internally, causing infinite redirect.

**Fix**: Edit domain in DokPloy, set HTTPS to OFF. Cloudflare handles HTTPS externally.

## 504 Gateway Timeout

### Traefik not on dokploy-network
**Symptom**: All Traefik-routed services return 504.

**Cause**: Traefik container isn't connected to `dokploy-network`.

**Fix**:
```bash
ssh user@arise-server "docker network connect dokploy-network dokploy-traefik"
```

Note: May need to re-run if Traefik is recreated.

## Container Won't Start

### Port conflict
**Symptom**: Deployment fails, logs show port already in use.

**Cause**: Another container is using the same host port.

**Fix**: Check existing ports:
```bash
ssh user@arise-server "docker ps --format '{{.Names}}: {{.Ports}}'"
```

Use a different port or use Traefik routing (no port mapping needed).

### Missing environment variables
**Symptom**: Container exits immediately, logs show missing config.

**Fix**: Check DokPloy → service → Environment. Add required variables.

## Debugging Commands

### Check container status
```bash
ssh user@arise-server "docker ps -a | grep myservice"
```

### View container logs
```bash
ssh user@arise-server "docker logs -f <container-name>"
```

### Check container networks
```bash
ssh user@arise-server "docker inspect <container-name> --format '{{json .NetworkSettings.Networks}}' | jq"
```

### Check Traefik routes
```bash
ssh user@arise-server "docker logs dokploy-traefik 2>&1 | grep myservice"
```

### Test from server
```bash
ssh user@arise-server "curl -I http://localhost:5678/"
```

## Related Docs

- [Networking](networking.md) - Understanding the network architecture
- [Deploying Services](deploying-services.md) - Correct deployment steps
