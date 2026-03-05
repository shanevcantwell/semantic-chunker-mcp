# Trajectory Analysis

Treats text as a particle moving through embedding space. Each sentence is a point; the path between them encodes rhetorical structure.

## Metrics

| Metric | Definition | What it measures |
|--------|-----------|-----------------|
| Velocity | `‖e[i+1] - e[i]‖` | Pacing — magnitude of semantic shift between sentences |
| Acceleration | `|v[i+1] - v[i]|` | Rhythm — rate of pacing change |
| Curvature | Angular deflection between consecutive displacement vectors | Direction change in full embedding space |

## Spike Detection

An acceleration spike fires when `a[i] >= threshold` (default 0.3). Each spike records:

- **Index**: Position in sentence sequence
- **Magnitude**: Raw acceleration value
- **Isolation score**: How much the spike stands out from neighbors
- **Position ratio**: Where in the passage it occurs (0.0 = start, 1.0 = end)

## Composite Scores

**Deadpan score** (0–1): Isolated acceleration spikes against a stable background. Few spikes, high isolation, low background noise, strong contrast.

```
deadpan = 0.25 × spikiness + 0.35 × mean_isolation + 0.20 × background_stability + 0.20 × contrast
```

**Heller score** (0–1): Circular structure with deceleration. High pairwise similarity, low net displacement, negative velocity trend.

```
heller = 0.35 × circularity + 0.40 × tautology_density + 0.25 × deceleration
```

## Comparison / Fitness

`compare_trajectories` scores how well one passage matches another's structure (lower = better):

- DTW on acceleration profiles
- Pearson correlation of interpolated acceleration
- Spike position and count matching
- Weighted toward absolute deadpan quality (30%)

| Score | Meaning |
|-------|---------|
| < 0.3 | Excellent match |
| 0.3–0.5 | Good, some rhythm deviation |
| 0.5–0.7 | Moderate |
| > 0.7 | Poor match |

## Context Window Smoothing

The Gradio UI supports a sliding context window that averages N consecutive sentence embeddings before computing metrics. This smooths out filler (verbal tics, short interjections) without re-embedding.

`smoothed[i] = mean(e[i], e[i+1], ..., e[i+w-1])`

Window size 1 = no smoothing (default).

## Visualizations (Gradio UI)

- **Trajectory profile**: 3-panel velocity/acceleration/curvature with spike markers
- **PCA 2D projection**: Sentence embeddings projected to 2D, colored by position
- **Cosine similarity heatmap**: NxN pairwise similarity, reveals topic blocks and repetitions
- **Comparison overlay**: Two-passage velocity/acceleration on normalized x-axis

## Known Limitation

Velocity collapses 4096D displacement to a scalar (L2 norm), discarding direction. Acceleration compounds this. The PCA and heatmap visualizations compensate by operating on the full embedding matrix.
