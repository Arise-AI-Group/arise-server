# User Guide

How to use the agent server for AI-assisted development.

## Connecting

```bash
# Via Tailscale (automatic after setup)
ssh user@agent-server
```

## Claude Code

Claude Code is the official Anthropic CLI for AI-assisted coding.

### First-Time Setup (OAuth Login)

Each user needs to authenticate once with their Anthropic subscription:

```bash
# SSH into the server
ssh user@agent-server

# Run claude - first time triggers OAuth
claude
```

This displays a URL like:
```
https://console.anthropic.com/oauth/device?code=XXXX
```

Open that URL in any browser (laptop/phone), approve the login. Credentials are saved to `~/.claude/` - you only need to do this once.

### Running Claude Code Sessions

**Interactive session with tmux** (recommended for long tasks):

```bash
# Start a new tmux session
tmux new -s project-name

# Run claude
claude

# Work with Claude...

# Detach (session keeps running)
Ctrl-A then D

# Later, reattach
tmux attach -t project-name
```

**Headless mode** (for automation):

```bash
# Run a specific task
claude --print "analyze this codebase" --output-format json

# Pipe input
echo "review this code" | claude --print
```

### tmux Basics

| Action | Keys |
|--------|------|
| Detach (keep running) | `Ctrl-A` then `D` |
| Split horizontal | `Ctrl-A` then `|` |
| Split vertical | `Ctrl-A` then `-` |
| Switch panes | `Ctrl-A` then arrow keys |
| List sessions | `tmux ls` |
| Attach to session | `tmux attach -t name` |
| Kill session | `tmux kill-session -t name` |

## Ollama (Local LLM)

Ollama runs locally for tasks that don't need Claude. The `qwen2.5:14b` model is pre-installed.

### CLI Usage

```bash
# Quick prompt
docker exec -it ollama ollama run qwen2.5:14b "Explain Docker in one sentence"

# Interactive chat
docker exec -it ollama ollama run qwen2.5:14b
```

### API Usage

```bash
# Generate completion
curl http://localhost:11434/api/generate -d '{
  "model": "qwen2.5:14b",
  "prompt": "Explain Docker in one sentence"
}'

# OpenAI-compatible endpoint
curl http://localhost:11434/v1/chat/completions -d '{
  "model": "qwen2.5:14b",
  "messages": [{"role": "user", "content": "Hello"}]
}'
```

### Model Management

```bash
# List installed models
curl http://localhost:11434/api/tags | jq

# Pull additional models
docker exec ollama ollama pull llama3.1:8b
docker exec ollama ollama pull qwen2.5:32b

# Remove a model
docker exec ollama ollama rm model-name
```

### Recommended Models

| Model | Size | RAM | Speed | Use Case |
|-------|------|-----|-------|----------|
| `llama3.1:8b` | 4.7GB | ~8GB | Fast | Quick tasks |
| `qwen2.5:14b` | 9GB | ~12GB | Medium | Coding + reasoning (default) |
| `qwen2.5:32b` | 20GB | ~25GB | Slower | High quality |
| `llama3.1:70b` | 40GB | ~45GB | Slow | Best quality |

## Workspaces

Each user has their own workspace:

```
/opt/agents/
├── trent/     # trent's workspace
├── matthew/   # matthew's workspace
└── shared/    # shared files (docker group access)
```

## Best Practices

1. **Use tmux for long-running tasks** - Sessions persist even if SSH disconnects

2. **One Claude session per tmux window** - Avoid running multiple Claude instances simultaneously

3. **Use Ollama for quick local tasks** - Saves API calls when Claude isn't needed

4. **Commit work frequently** - The server may need reboots for updates

5. **Check disk space periodically** - LLM models are large
   ```bash
   df -h
   docker system df
   ```
