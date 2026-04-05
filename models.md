# Model Recommendations

Recommended models for consumer GPU setups. Pull with `ollama pull <model>`.

## By Use Case

### General Chat / Assistant
| Model | Size | VRAM | Notes |
|-------|------|------|-------|
| `llama3.2:latest` | 3B | ~3GB | Best small model, fast |
| `llama3.2:11b` | 11B | ~8GB | Good balance |
| `llama3.1:8b` | 8B | ~6GB | Solid all-rounder |
| `llama3.1:70b` | 70B | ~45GB | Best quality (needs 2x3090 or similar) |
| `mistral:7b` | 7B | ~5GB | Fast, good instruction following |

### Coding
| Model | Size | VRAM | Notes |
|-------|------|------|-------|
| `codellama:7b` | 7B | ~5GB | Meta's code model |
| `qwen2.5-coder:7b` | 7B | ~5GB | Excellent for code, multilingual |
| `qwen2.5-coder:32b` | 32B | ~22GB | Best local coding model |
| `deepseek-coder-v2` | 16B | ~12GB | Strong competitor |

### Embeddings / RAG
| Model | Size | Notes |
|-------|------|-------|
| `nomic-embed-text` | 137M | Best local embedding model |
| `mxbai-embed-large` | 334M | High quality |

### Multimodal (Vision)
| Model | Size | VRAM | Notes |
|-------|------|-------|-------|
| `llava:7b` | 7B | ~6GB | Image understanding |
| `llama3.2-vision:11b` | 11B | ~9GB | Meta's vision model |

---

## GPU Memory Guide

### Single GPU
| VRAM | Recommended Max Model Size |
|------|---------------------------|
| 8GB  | 7B models (4-bit quantized) |
| 12GB | 13B models (4-bit) |
| 16GB | 13B full, 30B (4-bit) |
| 24GB | 30B full, 70B (4-bit) |

### Multi-GPU (e.g., 2x 24GB = 48GB total)
- Run 70B models in full precision
- Run multiple smaller models simultaneously
- Ollama handles multi-GPU automatically

### Quick Pulls
```bash
# Recommended starter set
ollama pull llama3.2
ollama pull qwen2.5-coder:7b
ollama pull nomic-embed-text
ollama pull mistral

# Larger models (if you have the VRAM)
ollama pull llama3.1:70b
ollama pull qwen2.5-coder:32b
```
