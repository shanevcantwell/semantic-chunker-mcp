# semantic-chunker

Embedding space analysis toolkit. Measures semantic drift between texts, traces trajectory dynamics through prose, and exposes everything as MCP tools for agentic integration.

## Quick Start

```bash
# Install
pip install -e .

# GPU support (NV-Embed-v2, ~14GB VRAM)
pip install -e ".[gpu]"

# Launch Gradio UI
python -m semantic_chunker

# Or start MCP server
semantic-chunker-mcp
```

### Docker

```bash
docker build -t semantic-chunker-mcp .
docker run -i --rm semantic-chunker-mcp
```

Or with docker-compose for host networking and data mounts:

```bash
docker-compose up
```

## Embedding Backends

Three interchangeable backends, selected via `EMBEDDING_BACKEND` environment variable:

| Backend | Model | Dimensions | Notes |
|---------|-------|------------|-------|
| `nv_embed` | NV-Embed-v2 | 4096 | GPU, fp16, highest quality |
| `lmstudio` | Any GGUF via OpenAI API | Varies | Local LM Studio server |
| `sentence_transformers` | Any HuggingFace model | Varies | General purpose |

Configure in `.env`:

```
EMBEDDING_BACKEND=nv_embed
```

## MCP Tools

8 tools over JSON-RPC (stdio). See [docs/tools.md](docs/tools.md) for full reference.

| Tool | Description |
|------|-------------|
| `embed_text` | Get embedding vector for text |
| `calculate_drift` | Cosine distance between two texts |
| `classify_document` | Similarity-based document classification |
| `analyze_trajectory` | Velocity, acceleration, curvature metrics for a passage |
| `compare_trajectories` | Fitness score: compare two passages structurally |
| `model_status` | Check embedding backend state |
| `model_load` | Load a specific backend |
| `model_unload` | Unload model and free memory |

### Configure in Claude Code

```json
{
  "mcpServers": {
    "semantic-chunker": {
      "command": "semantic-chunker-mcp",
      "env": {
        "EMBEDDING_BACKEND": "nv_embed"
      }
    }
  }
}
```

## Gradio UI

Two tabs:

- **Drift** — Pairwise cosine distance between texts
- **Trajectory** — Analyze single passages or compare two. Interactive Plotly visualizations: velocity/acceleration/curvature profiles, PCA 2D projection, cosine similarity heatmap. Adjustable acceleration threshold and context window smoothing.

```bash
python -m semantic_chunker
# Opens at http://localhost:7861
```

## Project Structure

```
semantic_chunker/
├── embeddings/        # NV-Embed-v2, LM Studio, SentenceTransformers adapters
├── mcp/
│   ├── server.py      # MCP entry point
│   ├── state_manager.py
│   └── commands/      # embeddings, classification, trajectory, model
├── ui/
│   ├── app.py         # Gradio application
│   ├── state.py       # Session state
│   └── tabs/          # drift, trajectory
└── utils/             # Text cleaning, HTML extraction
```

## Requirements

- Python 3.10+
- PyTorch 2.0+ (for NV-Embed-v2 backend)
- See `pyproject.toml` for full dependency list

## License

MIT
