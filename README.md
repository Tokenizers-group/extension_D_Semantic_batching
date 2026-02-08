# Chronos-2 Semantic Cross-Learning Extension (Inference-Time Grouping)

This repository contains an **inference-time extension layer** for **Chronos-2** that improves *cross-learning* by changing **only batch composition and group IDs**—the **Chronos-2 forward pass, tokenization, parameters, and inference path are unchanged**.

✅ **Extension location:**

src/chronos/chronos2/extension_4/

---

## What this extension does
Chronos-2 controls inference-time information sharing via **group IDs** within each batch:

- **Cross-learning OFF:** each series gets its own group ID → independent univariate inference.
- **Cross-learning ON:** multiple items share a group ID → the model can use shared context during the same (unchanged) forward pass.

**This extension modifies only how batches are formed:**  
for each **target** series, we optionally attach a small set of **helper** (auxiliary) series and assign them the **same group ID**, creating an **augmented cross-learning context** while keeping Chronos-2 tokenization, parameters, and inference path identical.

---

## Modes compared
All modes share the same evaluation configuration and differ **only** in how group mates (helpers) are chosen:

1. **Baseline**
   - Cross-learning OFF
   - No helpers

2. **Random Cross-Learning**
   - Cross-learning ON
   - Helpers sampled uniformly at random

3. **Semantic Cross-Learning (ours)**
   - Cross-learning ON
   - Helpers retrieved by **cosine similarity** between lightweight **series signatures** computed from history windows
   - Deterministic packing policy:
     - **Top-K first**
     - **weak fill**
     - **global fallback** (reduces fragmentation)

> ✅ In all cases, **only grouping and batching change**. The Chronos-2 forward pass is identical.

---

## Experimental setup
We evaluate on **FEV** tasks and **flatten** each task into a set of **univariate forecasting instances**:
regardless of whether a task is originally **univariate, covariate, or multivariate**, evaluation forecasts **a single target series per instance**.

- **Benchmark:** `fev-bench` (100 tasks; 80 used for paired comparisons after filtering)
- **Metrics:** `MASE`, `WQL` (lower is better)
- **Inference settings:** `amazon/chronos-2`, `float32`, `batch_size=32`
- **Semantic defaults:** `top_k=64`, `neighbor_threshold=0.20`
  - Optional clustering (if used): `num_clusters=50`, `kmeans_iters=25`

### Statistics
Random vs Semantic comparisons are **paired across tasks**:
- paired **t-test** (two-sided)
- **Wilcoxon signed-rank** (one-sided, H1: Semantic < Random)

---

## Results (FEV, n=80 paired tasks)
Semantic cross-learning shows a consistent *directional* advantage over Random:

- **MASE:** Semantic wins on **66.2%** of tasks; mean gain **+0.78%**
  - `p_t = 0.137`, `p_W = 0.0007`
- **WQL:** Semantic wins on **67.5%** of tasks; mean gain **+0.69%**
  - `p_t = 0.201`, `p_W = 0.0029`

**Interpretation:** improvements are systematic across tasks (Wilcoxon significant), but mean gains are small and heavy-tailed (t-test not significant).

Notes:
- Tasks with “exploded” MASE (MASE > 10 in any mode) are removed.
- 17/100 tasks could not be evaluated due to limited compute; paired comparisons use **n = 80**.
---

## Citation
If you use this work, please cite:
- Chronos-2: Ansari et al., 2025
- FEV: Shchur et al., 2025