# Agent Server

Ubuntu server for running long-lived AI agents with private SSH access via Tailscale.

## Quick Start

```bash
# SSH in (via Tailscale)
ssh trent@agent-server

# Start Claude Code session
tmux new -s work
claude

# Detach (keeps running)
Ctrl-A then D

# Reattach later
tmux attach -t work
```

## Access

| User | SSH Command |
|------|-------------|
| trent | `ssh trent@agent-server` |
| matthew | `ssh matthew@agent-server` |

## Services

| Service | Endpoint | Description |
|---------|----------|-------------|
| Ollama API | `http://localhost:11434` | Local LLM (qwen2.5:14b) |
| Claude Code | `claude` | AI coding assistant (OAuth login) |

## Documentation

- [SETUP.md](SETUP.md) - How to set up from scratch
- [USER-GUIDE.md](USER-GUIDE.md) - How to use Claude Code and Ollama
- [CONFIG.md](CONFIG.md) - Ports, paths, and configuration reference

## Server Specs

- **CPU**: 48 cores
- **RAM**: 128GB
- **OS**: Ubuntu 24.04 LTS
- **Network**: Tailscale (private access)
