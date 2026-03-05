# MCP Tools Reference

8 tools exposed over JSON-RPC (stdio) by `semantic-chunker-mcp`.

---

## Embedding Tools

### embed_text

Get embedding vector for text.

```json
{
  "text": "string (required)",
  "full_vector": "boolean (default: false)",
  "model": "string (optional, override backend model)"
}
```

Returns `embedding_preview` (first 10 dimensions) by default. Set `full_vector: true` for the complete vector.

---

### calculate_drift

Cosine distance between two texts.

```json
{
  "text_a": "string (required)",
  "text_b": "string (required)"
}
```

Returns `drift` (0.0–1.0+) and `interpretation`:

| Range | Meaning |
|-------|---------|
| 0.0–0.1 | Very similar |
| 0.1–0.3 | Related |
| 0.3–0.5 | Moderate divergence |
| 0.5–0.7 | Different semantics |
| 0.7+ | Unrelated |

---

## Classification Tools

### classify_document

Classify text by cosine similarity to category exemplars.

```json
{
  "content": "string (required, truncated to 2000 chars)",
  "categories": {
    "category-a": "Description or exemplar text for category A",
    "category-b": "Description or exemplar text for category B"
  },
  "threshold": "number (default: 0.85)"
}
```

Returns `best_match`, `similarity`, `confident` (boolean), and `all_similarities`.

---

## Trajectory Tools

### analyze_trajectory

Compute velocity, acceleration, and curvature for a text passage. Each sentence becomes a point in embedding space; metrics describe the path between them.

```json
{
  "text": "string (required, 2+ sentences)",
  "acceleration_threshold": "number (default: 0.3)",
  "include_sentences": "boolean (default: false)"
}
```

Returns:

| Field | Description |
|-------|-------------|
| `n_sentences` | Sentence count |
| `mean_velocity` | Average pacing between sentences |
| `velocity_variance` | Pacing consistency |
| `mean_acceleration` | Average rhythm change |
| `max_acceleration` | Largest pacing spike |
| `acceleration_spikes` | List of spikes above threshold, with position and isolation score |
| `deadpan_score` | Isolated spikes against calm background (0–1) |
| `heller_score` | Circular structure with deceleration (0–1) |
| `circularity_score` | Semantic looping (sentence i resembles sentence i-2) |
| `tautology_density` | High pairwise similarity + low net displacement |

---

### compare_trajectories

Compare two passages structurally. Returns a fitness score (lower = closer match).

```json
{
  "golden_text": "string (required)",
  "synthetic_text": "string (required)",
  "acceleration_threshold": "number (default: 0.3)"
}
```

Fitness components: DTW on acceleration profiles, Pearson correlation, spike position/count matching.

| Fitness | Meaning |
|---------|---------|
| < 0.3 | Excellent structural match |
| 0.3–0.5 | Good match, some rhythm deviation |
| 0.5–0.7 | Moderate — structure present but weak |
| > 0.7 | Poor match |

---

## Model Management

### model_status

Report current backend state: type, model name, dimensions, cache size.

No parameters.

---

### model_load

Load a specific embedding backend.

```json
{
  "backend": "nv_embed | lmstudio | sentence_transformers",
  "options": "object (optional backend-specific config)"
}
```

---

### model_unload

Unload current model and clear embedding cache. Frees GPU memory.

No parameters.

---

## Error Format

All tools return errors as:

```json
{
  "error": "Description of what went wrong"
}
```

## Architecture

```
semantic_chunker/mcp/
├── server.py           # Entry point, tool dispatch
├── state_manager.py    # Embedding cache, adapter lifecycle
└── commands/
    ├── embeddings.py   # embed_text, calculate_drift
    ├── classification.py
    ├── trajectory.py   # analyze_trajectory, compare_trajectories
    └── model.py        # model_status, model_load, model_unload
```
