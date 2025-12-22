# Configuration Reference

Quick reference for agent server configuration.

## Network

| Item | Value |
|------|-------|
| Hostname | `agent-server` (via Tailscale) |
| SSH | `ssh <user>@agent-server` |
| Ollama API | `http://localhost:11434` |

## Users

| User | Home | Workspace | Docker Access |
|------|------|-----------|---------------|
| trent | `/home/trent` | `/opt/agents/trent` | Yes |
| matthew | `/home/matthew` | `/opt/agents/matthew` | Yes |

## Directories

```
/opt/agents/
├── trent/              # trent's workspace (owner: trent)
├── matthew/            # matthew's workspace (owner: matthew)
└── shared/             # shared (owner: root:docker, mode 775)

/opt/ollama/
└── docker-compose.yml  # Ollama container config
```

## Services

| Service | Port | Restart Policy | Config Location |
|---------|------|----------------|-----------------|
| Docker | - | on-boot | systemd |
| Tailscale | - | on-boot | systemd |
| Ollama | 11434 | unless-stopped | `/opt/ollama/docker-compose.yml` |

## Docker

### Ollama Container

```yaml
# /opt/ollama/docker-compose.yml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama-data:/root/.ollama

volumes:
  ollama-data:
```

### Container Management

```bash
# Check status
docker ps

# Restart Ollama
cd /opt/ollama && docker compose restart

# View logs
docker logs -f ollama

# Resource usage
docker stats
```

## Installed Software

| Software | Version | Install Method |
|----------|---------|----------------|
| Ubuntu | 24.04 LTS | Base OS |
| Docker | 29.1.3 | Docker apt repo |
| Docker Compose | 5.0.0 | Docker plugin |
| Node.js | 20.19.6 | NodeSource apt repo |
| Claude Code | Latest | npm global |
| Ollama | Latest | Docker |
| Tailscale | Latest | Tailscale apt repo |

## tmux Configuration

```bash
# ~/.tmux.conf
Prefix: Ctrl-A (instead of Ctrl-B)
Mouse: Enabled
Window numbering: Starts at 1
Split horizontal: Ctrl-A |
Split vertical: Ctrl-A -
```

## Claude Code

| Item | Location |
|------|----------|
| Binary | `/usr/bin/claude` |
| Credentials | `~/.claude/` (per user) |
| Auth method | OAuth (Anthropic subscription) |

## Maintenance Commands

```bash
# System updates
sudo apt update && sudo apt upgrade -y

# Check services
sudo systemctl status docker
tailscale status
docker ps

# Disk usage
df -h
docker system df

# View Ollama logs
docker logs -f ollama

# Restart Ollama
cd /opt/ollama && docker compose restart
```

## Future: Cloudflare Tunnel (Not Yet Configured)

When webhooks are needed:

```yaml
# /etc/cloudflared/config.yml (template)
tunnel: <tunnel-id>
credentials-file: /home/trent/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: hooks.example.com
    service: http://localhost:5000
  - service: http_status:404
```
