# local-ai-stack

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)](https://docs.docker.com/compose/)
[![NVIDIA](https://img.shields.io/badge/GPU-NVIDIA-76B900)](https://developer.nvidia.com/cuda)

A production-ready Docker Compose stack for running local LLM infrastructure on consumer NVIDIA GPUs. Includes Ollama for model serving, Open WebUI for a ChatGPT-like interface, and an OpenAI-compatible proxy.

**Currently running Gemma 4 26B (MoE, 3.8B active params) on a single RTX 4060 8GB at ~9-11 tok/s.** Powers [WaldoNet AI](https://waldonet.simonlpaige.com), a neighborhood AI cooperative in Kansas City.

## Stack

| Service | Purpose | Port |
|---------|---------|------|
| [Ollama](https://ollama.com) | Model serving (GPU-accelerated) | 11434 |
| [Open WebUI](https://github.com/open-webui/open-webui) | Web chat interface | 3000 |

```
Your app / OpenAI SDK
        │
        ▼
  Ollama :11434        ← GPU inference
        │
        ├── gemma4:26b (primary — 26B MoE, 3.8B active)
        ├── nomic-embed-text (embeddings)
        └── (any Ollama-supported model)

  Open WebUI :3000     ← Web chat interface
        │
        └── Ollama :11434
```

## Real-World Performance (RTX 4060 8GB)

| Model | VRAM | Speed | Context | Notes |
|-------|------|-------|---------|-------|
| gemma4:26b | ~7.8GB (partial offload) | 9-11 tok/s | 128K | Primary model, MoE architecture |
| nomic-embed-text | ~300MB | instant | — | Embedding model for RAG |

> Gemma 4 26B is a Mixture-of-Experts model with only 3.8B active parameters per forward pass, making it remarkably capable on consumer hardware. Apache 2.0 licensed.

## Requirements

- Docker + Docker Compose (or native Ollama install)
- NVIDIA GPU with CUDA support
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) (Docker path)

> Works on consumer GPUs from RTX 3060 up. An 8GB card runs Gemma 4 26B comfortably. A 24GB card can run 70B quantized models.

## Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/simonlpaige/local-ai-stack.git
cd local-ai-stack

# 2. Configure environment
cp .env.example .env
# Edit .env — at minimum set WEBUI_SECRET_KEY

# 3. Start the stack
docker compose up -d

# 4. Pull Gemma 4
docker exec ollama ollama pull gemma4:26b

# 5. Open the UI
open http://localhost:3000
```

## Cloudflare Tunnel (Remote Access)

Expose your local stack to the internet securely:

```bash
# Install cloudflared
# Create a tunnel
cloudflared tunnel create my-ai-stack

# Route to Open WebUI
cloudflared tunnel route dns my-ai-stack ai.yourdomain.com

# Run it
cloudflared tunnel --url http://localhost:3000 run my-ai-stack
```

## Open WebUI Features

- Knowledge bases (upload documents for RAG)
- Model-specific system prompts and temperature presets
- Web search integration (DuckDuckGo)
- Multi-user with role-based access
- Conversation history and export

## Configuration

### Environment Variables

```bash
# Required
WEBUI_SECRET_KEY=your-long-random-secret

# Optional: Ollama connection (if not on same host)
OLLAMA_BASE_URL=http://localhost:11434
```

### Model Recommendations by VRAM

| VRAM | Recommended Models |
|------|-------------------|
| 6GB | gemma4:9b, phi3.5 |
| 8GB | gemma4:26b (MoE), llama3.2:8b |
| 12GB | gemma4:26b (full GPU), mistral-nemo |
| 24GB | llama3.3:70b-q4, deepseek-coder-v2 |

## Useful Commands

```bash
# Check GPU usage
nvidia-smi

# List loaded models
ollama list

# Pull a new model
ollama pull gemma4:26b

# Check running models
ollama ps

# View logs
docker compose logs -f
```

## License

MIT
