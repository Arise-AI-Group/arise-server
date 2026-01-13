# Architecture Overview

## Network Topology

```mermaid
flowchart TB
    subgraph tailscale["TAILSCALE NETWORK (Private/Maintenance)"]
        subgraph agent["agent-server"]
            claude["Claude Code<br/>(AI Assistant)"]
            ollama["Ollama<br/>qwen2.5:14b"]
            agent_spec["48 cores, 128GB RAM<br/>Ubuntu 24.04"]
        end

        subgraph arise["arise-server"]
            dokploy["DokPloy<br/>(Orchestrator)"]
            traefik["Traefik<br/>(Reverse Proxy)"]
            containers["Containers<br/>n8n, demos..."]
            dokploy --> traefik --> containers
        end
    end

    local["Local Machine"] -.->|SSH via Tailscale| agent
    local -.->|SSH via Tailscale| arise

    arise --> tunnel["Cloudflare Tunnel<br/>(TLS termination)"]
    tunnel --> internet["INTERNET<br/>*.arisegroup-tools.com"]
```

## Components

### Tailscale (Both Servers)
- **Purpose**: Private network for internal access
- **Use case**: SSH maintenance, inter-server communication
- **Hostnames**: `agent-server`, `arise-server`

### agent-server
| Component | Description |
|-----------|-------------|
| Claude Code | AI coding assistant (Anthropic OAuth) |
| Ollama | Local LLM server (qwen2.5:14b) |
| tmux | Session persistence for long-running agents |
| Docker | Container runtime for Ollama |

**Access**: Tailscale only (no public exposure)

### arise-server
| Component | Description |
|-----------|-------------|
| DokPloy | Container orchestration UI (port 3000) |
| Traefik | Reverse proxy, routes by hostname |
| Docker | Container runtime for all services |
| Cloudflare Tunnel | Public internet ingress |

**Access**: Tailscale (SSH) + Cloudflare (public web)

### Cloudflare
| Component | Description |
|-----------|-------------|
| DNS | Wildcard `*.arisegroup-tools.com` → tunnel |
| Tunnel | Secure connection from Cloudflare edge to server |
| TLS | Handles HTTPS termination (internal traffic is HTTP) |

## Traffic Flow

### Public Web Request

```mermaid
sequenceDiagram
    participant User as User Browser
    participant CF as Cloudflare Edge
    participant Server as arise-server
    participant Traefik
    participant Container as Container (n8n, demo)

    User->>CF: HTTPS request
    Note over CF: TLS termination
    CF->>Server: Tunnel (encrypted)
    Server->>Traefik: HTTP
    Traefik->>Container: HTTP (route by hostname)
    Container-->>User: Response
```

### SSH Access (Maintenance)

```mermaid
sequenceDiagram
    participant Local as Local Machine
    participant TS as Tailscale VPN
    participant Server as agent/arise-server

    Local->>TS: Connect
    TS->>Server: SSH
    Server-->>Local: Shell session
```

## Current Services

### arise-server
| Service | URL | Description |
|---------|-----|-------------|
| DokPloy | dokploy.arisegroup-tools.com | Container management UI |
| n8n (prod) | n8n.arisegroup-tools.com | Production workflow automation |
| tc-n8n | tc-n8n.arisegroup-tools.com | Trent's sandbox |
| ca-n8n | ca-n8n.arisegroup-tools.com | CA's sandbox |
| ma-n8n | ma-n8n.arisegroup-tools.com | MA's sandbox |
| mk-n8n | mk-n8n.arisegroup-tools.com | MK's sandbox |

### agent-server
| Service | Endpoint | Description |
|---------|----------|-------------|
| Ollama | localhost:11434 | Local LLM API |
| Claude Code | CLI (`claude`) | AI coding assistant |
