# arise-server

Container hosting server for the Arise team: DokPloy, n8n, demos, and client sandboxes with public access via Cloudflare.

## Quick Access

| Resource | URL/Command |
|----------|-------------|
| SSH | `ssh user@arise-server` (via Tailscale) |
| DokPloy UI | https://dokploy.arisegroup-tools.com |
| Production n8n | https://n8n.arisegroup-tools.com |
| Wildcard domain | `*.arisegroup-tools.com` |

## Project Structure

```
arise-server/
├── CLAUDE.md                    # AI agent instructions
├── README.md                    # This file
├── infrastructure-registry.yaml # Service registry & port assignments
├── notes/                       # Working documentation
├── docs/                        # Formal documentation
│   ├── architecture.md
│   └── arise-server/
├── compose/                     # Active compose files
└── templates/                   # Docker compose templates
```

## Architecture

```
                                  ┌─────────────────┐
                                  │  arise-server   │
                                  │  (Hetzner)      │
                                  │                 │
                                  │  • DokPloy      │
                                  │  • Traefik      │
                                  │  • n8n          │
                                  │  • Sandboxes    │
                                  └────────┬────────┘
                                           │
                              ┌────────────┼────────────┐
                              │            │            │
                     ┌────────▼────────┐   │   ┌───────▼───────┐
                     │   Tailscale     │   │   │  Cloudflare   │
                     │   (SSH access)  │   │   │  Tunnel       │
                     └─────────────────┘   │   └───────┬───────┘
                                           │           │
                                           │   ┌───────▼───────┐
                                           │   │   INTERNET    │
                                           │   │ *.arisegroup  │
                                           │   │  -tools.com   │
                                           │   └───────────────┘
```

## Documentation

### Quick Start
- **[CHEATSHEET](docs/arise-server/CHEATSHEET.md)** - Single-page deployment reference (start here)

### Detailed Guides
- [Overview](docs/arise-server/overview.md) - Server specs and services
- [DokPloy](docs/arise-server/dokploy.md) - Container orchestration UI
- [Networking](docs/arise-server/networking.md) - Docker networks, Traefik, Cloudflare
- [Deploying Services](docs/arise-server/deploying-services.md) - Step-by-step guide
- [Troubleshooting](docs/arise-server/troubleshooting.md) - Common issues and fixes

### Reference
- [Architecture](docs/architecture.md) - Full system diagram

### Templates
- [Single-container compose](templates/single-container.yaml) - For sandboxes
- [Multi-container compose](templates/multi-container.yaml) - For production services

## Related

- **agent-server** - AI agents server (separate repo)
