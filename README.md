# Arise Infrastructure

Documentation for the Arise team infrastructure: two servers connected via Tailscale, with public access through Cloudflare.

## Quick Access

| Server | Purpose | SSH Access | Public URL |
|--------|---------|------------|------------|
| **arise-server** | Container hosting (DokPloy, n8n, demos) | `ssh trent@arise-server` | `*.arisegroup-tools.com` |
| **agent-server** | AI agents (Claude Code, Ollama) | `ssh trent@agent-server` | None (private) |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      TAILSCALE NETWORK                       │
│                    (Internal / Maintenance)                  │
│                                                             │
│   ┌─────────────────┐           ┌─────────────────┐        │
│   │  agent-server   │           │  arise-server   │        │
│   │  (AI Agents)    │           │  (Containers)   │        │
│   │                 │           │                 │        │
│   │  • Claude Code  │           │  • DokPloy      │        │
│   │  • Ollama       │           │  • Traefik      │        │
│   │  • tmux         │           │  • n8n, demos   │        │
│   └─────────────────┘           └────────┬────────┘        │
│          ↑                               │                  │
│          │ SSH                           │                  │
└──────────┼───────────────────────────────┼──────────────────┘
           │                               │
      (private)                   ┌────────▼────────┐
                                  │ Cloudflare      │
                                  │ Tunnel          │
                                  └────────┬────────┘
                                           │
                                  ┌────────▼────────┐
                                  │   INTERNET      │
                                  │  (*.arisegroup  │
                                  │   -tools.com)   │
                                  └─────────────────┘
```

- **Tailscale**: Private network for SSH access and maintenance
- **Cloudflare Tunnel**: Public internet access to arise-server services only

## Documentation

### [Architecture Overview](docs/architecture.md)
Full system diagram and component descriptions.

### arise-server (Container Hosting)
- [Overview](docs/arise-server/overview.md) - Server specs and services
- [DokPloy](docs/arise-server/dokploy.md) - Container orchestration UI
- [Networking](docs/arise-server/networking.md) - Docker networks, Traefik, Cloudflare
- [Deploying Services](docs/arise-server/deploying-services.md) - Step-by-step guide
- [Troubleshooting](docs/arise-server/troubleshooting.md) - Common issues and fixes

### agent-server (AI Agents)
- [Overview](docs/agent-server/overview.md) - Server specs and services
- [Setup](docs/agent-server/setup.md) - How to recreate from scratch
- [User Guide](docs/agent-server/user-guide.md) - Daily usage (Claude Code, tmux)
- [Config Reference](docs/agent-server/config.md) - Ports, paths, commands

### Templates
- [Single-container compose](templates/single-container.yaml) - For sandboxes
- [Multi-container compose](templates/multi-container.yaml) - For production services
