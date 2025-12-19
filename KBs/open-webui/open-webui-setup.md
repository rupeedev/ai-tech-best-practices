# Open WebUI - Local Setup Guide

Open WebUI is a self-hosted, feature-rich web interface for running LLMs locally with Ollama.

## Quick Start (Docker - Recommended)

### Option 1: Open WebUI + Ollama (All-in-One)

```bash
docker run -d \
  -p 3030:8080 \
  -v ollama:/root/.ollama \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:ollama
```

### Option 2: Open WebUI Only (Connect to Existing Ollama)

```bash
docker run -d \
  -p 3030:8080 \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

## Access

- **URL**: http://localhost:3030
- **First visit**: Create admin account

## Pull Models

### Via CLI

```bash
# Pull models inside container
docker exec -it open-webui ollama pull llama3.2
docker exec -it open-webui ollama pull mistral
docker exec -it open-webui ollama pull codellama
docker exec -it open-webui ollama pull phi3

# List available models
docker exec -it open-webui ollama list
```

### Via Web UI

1. Go to Settings (gear icon)
2. Navigate to Models
3. Enter model name and click Pull

## Popular Models

| Model | Size | Best For |
|-------|------|----------|
| `llama3.2` | 2GB | General chat |
| `llama3.2:1b` | 1.3GB | Lightweight |
| `mistral` | 4GB | Balanced performance |
| `codellama` | 4GB | Code generation |
| `phi3` | 2GB | Fast responses |
| `deepseek-coder` | 4GB | Coding tasks |

## Docker Compose Setup

```yaml
version: '3.8'

services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:ollama
    container_name: open-webui
    ports:
      - "3030:8080"
    volumes:
      - ollama:/root/.ollama
      - open-webui:/app/backend/data
    restart: always

volumes:
  ollama:
  open-webui:
```

Save as `docker-compose.yml` and run:

```bash
docker-compose up -d
```

## Management Commands

```bash
# Start
docker start open-webui

# Stop
docker stop open-webui

# Restart
docker restart open-webui

# View logs
docker logs -f open-webui

# Remove container
docker rm -f open-webui

# Remove container + data
docker rm -f open-webui
docker volume rm ollama open-webui
```

## GPU Support (NVIDIA)

```bash
docker run -d \
  -p 3030:8080 \
  --gpus all \
  -v ollama:/root/.ollama \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:ollama
```

## Connect to External APIs

Open WebUI can also connect to:
- OpenAI API
- Claude API
- Azure OpenAI
- Any OpenAI-compatible API

Configure in: Settings → Connections

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OLLAMA_BASE_URL` | Ollama server URL | `http://localhost:11434` |
| `OPENAI_API_KEY` | OpenAI API key | - |
| `WEBUI_AUTH` | Enable authentication | `true` |
| `WEBUI_NAME` | Custom UI name | `Open WebUI` |

## Troubleshooting

### Port Already in Use

```bash
# Use different port
docker run -d -p 3031:8080 ...
```

### Container Won't Start

```bash
# Check logs
docker logs open-webui

# Remove and recreate
docker rm -f open-webui
docker run -d ...
```

### Models Not Loading

```bash
# Check Ollama is running
docker exec -it open-webui ollama list

# Pull model again
docker exec -it open-webui ollama pull llama3.2
```

## Resources

- GitHub: https://github.com/open-webui/open-webui
- Docs: https://docs.openwebui.com
- Ollama Models: https://ollama.com/library
