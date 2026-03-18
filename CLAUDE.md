# Project: arise-server

> Owner: Trent (shared infrastructure for 4.0 Hero and Arise network)
> Status: Active

## Overview

Container hosting server managed by Trent, running on Hetzner with public access via Cloudflare:
- **DokPloy** for container orchestration
- **Traefik** for reverse proxy routing
- **n8n** for workflow automation (production + sandboxes)
- **Demo deployments** for client projects

This repository serves as the source of truth for:
- Server configuration and networking
- Service deployment procedures
- Docker compose templates
- Service registry (URLs, ports, routing)

## Goals

- Maintain accurate, up-to-date server documentation
- Provide repeatable deployment templates
- Track all services and their connectivity
- Enable contractors to deploy sandboxes and demos when engaged

---

## Tools

This project uses shared tools available via Claude Code plugins and MCP servers.

### Available Tools

| Tool | Access | Use for |
|------|--------|---------|
| Cloudflare API | Via `infrastructure` plugin (`/infrastructure:skills`) | DNS records, tunnel routes |
| DokPloy API | Via `infrastructure` plugin or MCP server | Compose deployment |

### MCP Servers

| Server | Purpose |
|--------|---------|
| dokploy | Container management via MCP |

### Tool Preferences

- Preferred diagram format: Mermaid (renders in GitHub/IDEs, easy to maintain)
- All infrastructure changes should be documented in this repo

---

## Templates

### Docker Compose Templates

| Template | Use Case | Location |
|----------|----------|----------|
| Single-container | Sandboxes, simple demos | `./templates/single-container.yaml` |
| Multi-container | Production with DB/cache | `./templates/multi-container.yaml` |

### Documentation Templates

- Server overview: Follow pattern in `./docs/arise-server/overview.md`

---

## Formatting & Style

### Writing Style

- Tone: Technical, concise
- Audience: Engineers and technically-capable team members
- Length preference: Concise with tables for reference data

### Documentation Style

- Use tables for quick reference (ports, URLs, commands)
- Include architecture diagrams as Mermaid (fenced with ```mermaid)
- Always document the "why" alongside the "what"

### Naming Conventions

- Files: `kebab-case.md`
- Folders: `lowercase/`
- Branches: `feat/description` or `fix/description`
- Services: `{initials}-{service}` for sandboxes (e.g., `tc-n8n`)

---

## Context

### Key Decisions

1. **Traefik for new services** - Use hostname-based routing via dokploy-network
2. **Direct tunnel routes for legacy** - Existing services with specific port mappings
3. **SQLite for sandboxes** - PostgreSQL only for production

### Constraints

- Always set HTTPS: OFF in DokPloy domains (Cloudflare handles TLS)
- Redeploy required after adding/modifying domains
- DokPloy admin role bug: must use API to enable permissions

### Quick Reference

| Resource | Location |
|----------|----------|
| Service registry | `./infrastructure-registry.yaml` |
| Deployment cheatsheet | `./docs/arise-server/CHEATSHEET.md` |
| Networking deep-dive | `./docs/arise-server/networking.md` |

### Important Links

- [DokPloy UI](https://dokploy.arisegroup-tools.com)
- [Production n8n](https://n8n.arisegroup-tools.com)
- GitHub: arise-server repo

---

## Session Checklist

Before starting work:
1. Check `./infrastructure-registry.yaml` for current state
2. Review `./notes/` for recent context
3. Read relevant docs in `./docs/arise-server/`

After completing work:
1. Update `./infrastructure-registry.yaml` if services changed
2. Add notes to `./notes/` for significant changes
3. Update relevant docs if procedures changed
4. Commit with descriptive message following branch conventions
