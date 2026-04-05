# local-ai-stack

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)](https://docs.docker.com/compose/)
[![NVIDIA](https://img.shields.io/badge/GPU-NVIDIA-76B900)](https://developer.nvidia.com/cuda)

A production-ready Docker Compose stack for running local LLM infrastructure on consumer NVIDIA GPUs. Includes Ollama for model serving, Open WebUI for a ChatGPT-like interface, and LiteLLM as an OpenAI-compatible proxy.

## Stack

| Service | Purpose | Port |
|---------|---------|------|
| [Ollama](https://ollama.com) | Model serving (GPU-accelerated) | 11434 |
| [Open WebUI](https://github.com/open-webui/open-webui) | Web chat interface | 3000 |
| [LiteLLM](https://github.com/BerriAI/litellm) | OpenAI-compatible proxy | 4000 |

```
Your app / OpenAI SDK
        │
        ▼
  LiteLLM :4000          ← OpenAI-compatible API
        │
        ├── ollama/llama3.2
        ├── ollama/mistral
        ├── ollama/qwen2.5-coder
        └── (optional cloud: gpt-4o, claude, etc.)
        
  Open WebUI :3000       ← Web chat interface
        │
        └── Ollama :11434 ← GPU inference
```

## Requirements

- Docker + Docker Compose
- NVIDIA GPU with CUDA support
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

> Works great on consumer GPUs (RTX 3060, 3090, 4090). A 24GB card can run 13B models in full precision or 70B models quantized. Two 24GB cards (e.g., dual RTX 3090) can run 70B models comfortably.

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

# 4. Pull your first model
docker exec ollama ollama pull llama3.2

# 5. Open the UI
open http://localhost:3000
```

## Configuration

### Environment Variables

Copy `.env.example` to `.env` and configure:

```bash
# Required
WEBUI_SECRET_KEY=your-long-random-secret

# LiteLLM API key (used by your apps to talk to the proxy)
LITELLM_MASTER_KEY=sk-your-key

# Optional: add cloud models alongside local ones
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
```

### Adding/Removing Models

Edit `litellm-config.yaml` to control which models the proxy exposes. Pull models with:

```bash
docker exec ollama ollama pull <model-name>
docker exec ollama ollama list
```

See [models.md](models.md) for recommendations by use case and VRAM budget.

## Using the LiteLLM Proxy

The proxy at `:4000` speaks the OpenAI API. Any OpenAI-compatible tool works:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:4000",
    api_key="sk-your-local-key"  # matches LITELLM_MASTER_KEY
)

response = client.chat.completions.create(
    model="llama3.2",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

```bash
# Or with curl
curl http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-local-key" \
  -H "Content-Type: application/json" \
  -d '{"model": "llama3.2", "messages": [{"role": "user", "content": "Hi"}]}'
```

## Multi-GPU Setup

Ollama automatically detects and uses all available NVIDIA GPUs. For best multi-GPU performance:

```yaml
# In docker-compose.yml, ollama service:
environment:
  - OLLAMA_NUM_PARALLEL=8      # increase for more GPUs
  - OLLAMA_MAX_LOADED_MODELS=3 # keep multiple models warm
```

## Useful Commands

```bash
# Check what's running
docker compose ps

# View logs
docker compose logs -f ollama
docker compose logs -f litellm

# Pull a new model
docker exec ollama ollama pull mistral

# Check GPU usage
nvidia-smi

# Stop everything
docker compose down

# Full reset (removes model data)
docker compose down -v
```

## Architecture Notes

- **Ollama** handles all GPU inference. It supports automatic GPU layer splitting across multiple GPUs.
- **Open WebUI** connects directly to Ollama for its chat UI but also supports external OpenAI/Anthropic APIs.
- **LiteLLM** acts as a smart router — your apps point to one endpoint and it handles routing to local or cloud models based on `litellm-config.yaml`.

## Tech Stack

- **Ollama** — local model runner (supports GGUF, safetensors)
- **Open WebUI** — feature-rich chat UI with RAG, tools, and multimodal support
- **LiteLLM** — OpenAI-compatible proxy with load balancing and fallbacks
- **Docker Compose** — orchestration
- **NVIDIA Container Toolkit** — GPU passthrough to containers

## License

MIT — see [LICENSE](LICENSE)
