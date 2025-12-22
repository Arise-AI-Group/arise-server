# Setup Guide

How to recreate this server from scratch.

## Prerequisites

- Ubuntu Server 24.04 LTS installed
- SSH enabled
- User account created
- Tailscale connected

## Phase 1: System Essentials

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essentials
sudo apt install -y git curl wget htop tmux build-essential

# Set hostname
sudo hostnamectl set-hostname agent-server
```

## Phase 2: Docker

```bash
# Add Docker GPG key and repo
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group
sudo usermod -aG docker $USER

# Verify
docker --version
docker compose version
```

## Phase 3: Additional Users

```bash
# Create user
sudo adduser --disabled-password --gecos '' matthew
echo 'matthew:TempPass123!' | sudo chpasswd

# Add to docker group
sudo usermod -aG docker matthew
```

## Phase 4: Directory Structure

```bash
# Create workspaces
sudo mkdir -p /opt/agents/{trent,matthew,shared}
sudo chown trent:trent /opt/agents/trent
sudo chown matthew:matthew /opt/agents/matthew
sudo chown root:docker /opt/agents/shared
sudo chmod 775 /opt/agents/shared

# Verify
ls -la /opt/agents/
```

## Phase 5: Node.js and Claude Code

```bash
# Add Node.js 20.x repo
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Verify
node --version
npm --version

# Install Claude Code globally
sudo npm install -g @anthropic-ai/claude-code

# Verify
which claude
```

## Phase 6: Ollama (Local LLM)

```bash
# Create Ollama directory
sudo mkdir -p /opt/ollama
sudo chown root:docker /opt/ollama

# Create docker-compose.yml
cat > /opt/ollama/docker-compose.yml << 'EOF'
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
EOF

# Start container
cd /opt/ollama && docker compose up -d

# Wait for startup
sleep 5

# Pull model (~9GB download)
docker exec ollama ollama pull qwen2.5:14b

# Verify
curl -s http://localhost:11434/api/tags | jq
```

## Phase 7: tmux Configuration

```bash
# Create config for trent
cat > ~/.tmux.conf << 'EOF'
set -g prefix C-a
unbind C-b
bind C-a send-prefix
set -g mouse on
set -g base-index 1
bind | split-window -h
bind - split-window -v
set -g default-terminal "screen-256color"
EOF

# Copy to matthew
sudo cp ~/.tmux.conf /home/matthew/.tmux.conf
sudo chown matthew:matthew /home/matthew/.tmux.conf
```

## Phase 8: Claude Code Authentication

Each user needs to authenticate with their Anthropic subscription:

```bash
# SSH in as the user
ssh user@agent-server

# Run claude
claude

# Opens OAuth URL - approve in browser
# Credentials saved to ~/.claude/
```

## Post-Setup

### Verify Everything

```bash
# Docker
docker ps

# Ollama
curl http://localhost:11434/api/tags | jq

# Claude Code
which claude

# Tailscale
tailscale status
```

### Reboot (if kernel was upgraded)

```bash
sudo reboot
```

### After Reboot

```bash
# Docker and Ollama auto-restart
docker ps

# Tailscale reconnects automatically
tailscale status
```

## Future: Cloudflare Tunnel

When webhooks are needed, set up Cloudflare Tunnel:

1. Install cloudflared on laptop
2. Run `cloudflared tunnel login`
3. Copy cert to server
4. Create tunnel: `cloudflared tunnel create agent-server`
5. Configure `/etc/cloudflared/config.yml`
6. Start service: `sudo cloudflared service install`
