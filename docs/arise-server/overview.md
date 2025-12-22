# arise-server Overview

Container hosting server for team demos, n8n instances, and production services.

## Access

| Method | Command/URL |
|--------|-------------|
| SSH | `ssh trent@arise-server` |
| DokPloy UI | https://dokploy.arisegroup-tools.com |

## Server Specs

- **Provider**: Hetzner (or similar)
- **OS**: Ubuntu 24.04 LTS
- **Network**: Tailscale (private) + Cloudflare Tunnel (public)

## Services Running

| Service | Purpose | Port |
|---------|---------|------|
| DokPloy | Container orchestration UI | 3000 |
| Traefik | Reverse proxy (managed by DokPloy) | 80 |
| Docker | Container runtime | - |
| Cloudflared | Cloudflare tunnel client | - |

## Container Projects

### Production
- **n8n-production**: Main workflow automation (PostgreSQL + Redis + workers)

### Sandboxes
- **tc-n8n**: Trent's sandbox
- **ca-n8n**: CA's sandbox
- **ma-n8n**: MA's sandbox
- **mk-n8n**: MK's sandbox

### Demos
- Various client demos deployed as needed

## Key URLs

| Service | URL |
|---------|-----|
| DokPloy | dokploy.arisegroup-tools.com |
| Production n8n | n8n.arisegroup-tools.com |
| Sandboxes | {initials}-n8n.arisegroup-tools.com |

## Related Docs

- [DokPloy Usage](dokploy.md)
- [Networking](networking.md)
- [Deploying Services](deploying-services.md)
- [Troubleshooting](troubleshooting.md)
