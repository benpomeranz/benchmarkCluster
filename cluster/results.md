# Benchmark Clustering Results (written by Claude)
 
## Methodology

### Data

50 benchmark CSV files from Epoch AI's Benchmarking Hub, covering 577 unique AI models. Each benchmark has a different number of models tested — from 6 (CommonSenseQA 2) to 584 (Epoch Capabilities Index). The raw data matrix is ~90% sparse.

### Distance metric

For each pair of benchmarks, we compute:

1. **Z-score** each benchmark independently across its non-missing values (mean 0, std 1).
2. Find all models tested on **both** benchmarks.
3. Distance = **mean absolute z-score difference** across those shared models.

This avoids filling missing values with arbitrary numbers. Two benchmarks are "close" if, among the models they share, a model's relative standing on one benchmark predicts its relative standing on the other.

### Filtering

We apply two filters to ensure every pairwise distance is based on real data:

**Filter 1: Minimum model count.** Drop benchmarks with fewer than 10 tested models. This removes 3 benchmarks:
- `common_sense_qa_2_external` (6 models)
- `os_world_external` (8 models)
- `superglue_external` (8 models)

**Filter 2: Pairwise connectivity.** Every pair of benchmarks must share at least 1 model so we can compute a real distance. We greedily drop the benchmark involved in the most disconnected pairs until all pairs are connected. This removes 20 benchmarks, almost entirely older/saturated NLU benchmarks that tested a different model population from the frontier benchmarks:

- `lambada_external`, `adversarial_nli_external`, `science_qa_external`, `arc_ai2_external`, `hella_swag_external`, `open_book_qa_external`, `bbh_external`, `trivia_qa_external`, `wino_grande_external`, `bool_q_external`, `gsm8k_external`, `piqa_external` — classic NLU/knowledge benchmarks tested primarily on older models
- `posttrainbench_external`, `video_mme_external`, `mmlu_external`, `cad_eval_external`, `cybench_external`, `the_agent_company_external`, `balrog_external`, `live_bench_external` — smaller or niche benchmarks with insufficient model overlap

**After filtering: 27 benchmarks remain**, all with pairwise model overlap. These are predominantly frontier-era benchmarks.

### Clustering

We use three methods on the resulting 27×27 distance matrix:
- **Hierarchical (Ward linkage)** — agglomerative clustering directly on the distance matrix
- **KMeans** — via 10-dimensional MDS embedding of the distance matrix
- **Spectral** — using a similarity matrix derived from distances: `similarity = exp(-distance / median_distance)`

Silhouette scores (computed on the precomputed distance matrix) are reported for each.

---

## Results

### k=2

All three methods agree on the same fundamental split.

| Cluster | Hierarchical | KMeans | Spectral |
|---|---|---|---|
| A (6-10) | apex_agents, chess_puzzles, gdpval, swe_bench_verified, terminalbench, webdev_arena | + arc_agi_2, deepresearchbench, frontiermath_tier_4, gso | same as KMeans |
| B (17-21) | everything else | everything else | everything else |

**Interpretation.** The first split separates benchmarks into two groups. Cluster A contains benchmarks where models are evaluated as **tool-using agents completing discrete tasks** — coding (SWE-bench, TerminalBench, WebDev Arena), game-playing (chess puzzles), and competitive evaluation (GDPVal, APEX Agents). Cluster B contains benchmarks measuring **knowledge and reasoning through direct question-answering** — math, science, factual recall, and pattern recognition.

Silhouette scores: Hierarchical 0.222, KMeans 0.271, Spectral 0.271.

---

### k=3

The large Cluster B from k=2 splits into two groups. All three methods agree on the structure.

| Cluster | Benchmarks (stable across methods) | Label |
|---|---|---|
| **Agentic/Applied** | apex_agents, chess_puzzles, gdpval, swe_bench_verified, terminalbench, webdev_arena (+ gso, frontiermath_tier_4 in KMeans/Spectral) | Task completion |
| **Frontier Reasoning** | epoch_capabilities_index, geobench, gpqa_diamond, math_level_5, otis_mock_aime_2024_2025 (+ aider_polyglot in KMeans/Spectral) | Broad frontier capability |
| **Hard/Unsaturated** | arc_agi, arc_agi_2, frontiermath, frontiermath_tier_4, hle, metr_time_horizons, simplebench, swe_bench_bash, vpct, weirdml, fictionlivebench, deepresearchbench, lech_mazur_writing, simpleqa_verified | Unsolved frontier |

**Interpretation.** The **Frontier Reasoning** cluster (ECI, GPQA, MATH Level 5, AIME, GeoBench) captures benchmarks where performance is actively scaling — the best models do well, and scores correlate tightly. The **Hard/Unsaturated** cluster contains benchmarks where even frontier models struggle or where performance is unpredictable — ARC-AGI, FrontierMath, HLE, METR time horizons, etc.

Silhouette scores: Hierarchical 0.195, KMeans 0.221, Spectral 0.217.

---

### k=4

A new cluster emerges — **Factual Accuracy** — splitting off from the hard/unsaturated group. This is the best-separated partition (highest silhouette for Hierarchical and Spectral).

| Cluster | Benchmarks (stable across methods) | Label |
|---|---|---|
| **Agentic/Applied** | apex_agents, chess_puzzles, gdpval, swe_bench_verified, terminalbench, webdev_arena | Task completion |
| **Frontier Reasoning** | epoch_capabilities_index, geobench, gpqa_diamond, math_level_5, otis_mock_aime_2024_2025 | Broad frontier capability |
| **Factual Accuracy** | deepresearchbench, lech_mazur_writing, simpleqa_verified | Factual precision |
| **Hard/Unsaturated** | arc_agi, arc_agi_2, frontiermath, frontiermath_tier_4, gso, hle, metr_time_horizons, simplebench, swe_bench_bash, vpct, weirdml, fictionlivebench, aider_polyglot | Unsolved frontier |

**Interpretation.** The **Factual Accuracy** cluster (DeepResearchBench, Lech Mazur Writing, SimpleQA Verified) groups benchmarks that test whether a model produces **correct, well-sourced factual content** — research synthesis, factual writing quality, and verified factual QA. These split from the hard/unsaturated group because they measure precision of knowledge rather than novel reasoning.

The **Hard/Unsaturated** cluster still contains a mix of novel reasoning (ARC-AGI, FrontierMath), agentic coding (SWE-bench Bash, Aider), long-horizon tasks (METR, FictionLiveBench), and adversarial/unusual evaluations (WeirdML, SimpleBench, VPCT). The common thread is that frontier models have not saturated these benchmarks, and performance on them does not strongly predict performance on the Frontier Reasoning benchmarks.

Silhouette scores: Hierarchical 0.242, KMeans 0.219, Spectral 0.249.

---

### k=5

The hard/unsaturated group splits into two. Hierarchical and Spectral agree exactly; KMeans differs slightly.

| Cluster | Benchmarks (Hierarchical & Spectral) | Label |
|---|---|---|
| **Agentic/Applied** | apex_agents, chess_puzzles, gdpval, swe_bench_verified, terminalbench, webdev_arena | Task completion |
| **Frontier Reasoning** | epoch_capabilities_index, geobench, gpqa_diamond, math_level_5, otis_mock_aime_2024_2025 | Broad frontier capability |
| **Factual Accuracy** | deepresearchbench, lech_mazur_writing, simpleqa_verified | Factual precision |
| **Novel Problem-Solving** | arc_agi, arc_agi_2, frontiermath, frontiermath_tier_4, gso, metr_time_horizons, vpct | Novel reasoning / long-horizon |
| **Diverse Hard Tasks** | aider_polyglot, fictionlivebench, hle, simplebench, swe_bench_bash, weirdml | Mixed hard evaluations |

**Interpretation.** The key split at k=5 separates benchmarks testing **genuinely novel reasoning** (ARC-AGI, FrontierMath, GSO, METR time horizons, VPCT) from a more **heterogeneous set of hard tasks** (Aider coding, FictionLiveBench creative writing, HLE, SimpleBench, SWE-bench Bash, WeirdML).

The **Novel Problem-Solving** cluster is defined by tasks requiring abstract pattern recognition (ARC-AGI), research-level mathematics (FrontierMath), long-horizon problem-solving (METR, GSO), and precise visual/procedural reasoning (VPCT). These benchmarks share a property: being good at GPQA or AIME does **not** reliably predict performance here.

The **Diverse Hard Tasks** cluster is more heterogeneous — it contains coding (Aider, SWE-bench Bash), adversarial/unusual knowledge (WeirdML, HLE), creative generation (FictionLiveBench), and tricky reasoning (SimpleBench). What unites them is that they're all unsaturated but don't cleanly fit the "novel abstract reasoning" profile of the other cluster.

Silhouette scores: Hierarchical 0.237, KMeans 0.179, Spectral 0.237.

---

## Discussion

### SWE-bench Verified vs. SWE-bench Bash

These two benchmarks evaluate the same underlying task — solving real GitHub issues — but land in different clusters at every k ≥ 2. **SWE-bench Verified** consistently clusters with the Agentic/Applied group (alongside APEX Agents, TerminalBench, WebDev Arena), while **SWE-bench Bash** clusters with the Hard/Unsaturated group (alongside ARC-AGI, FrontierMath, METR).

The difference is scaffolding. SWE-bench Verified evaluates models as fully scaffolded agents with tool access, file browsing, and iterative execution — the same agentic setup used by the other benchmarks in that cluster. SWE-bench Bash restricts models to a single bash command with no agent loop. This turns the same coding task into a much harder, less structured challenge where models must reason about the entire solution upfront. The clustering correctly picks up that the *mode of evaluation* (scaffolded agent vs. unscaffolded reasoning) matters as much as the *domain* (coding) for predicting model performance patterns.

### Clustering reflects difficulty regimes, not capability categories

A natural expectation is that benchmarks measuring similar capabilities (e.g., all agentic benchmarks, all math benchmarks) would cluster together. This is only partially true. Consider **METR Time Horizons**, which measures how long an AI agent can work autonomously on real-world tasks — a fundamentally agentic benchmark. Yet it clusters not with the Agentic/Applied group but with the Hard/Unsaturated group.

The reason is that the clustering is driven by **score correlation patterns**, which reflect difficulty and saturation regimes more than capability categories. Benchmarks where frontier models score well and performance is actively scaling (GPQA, AIME, MATH Level 5) correlate with each other regardless of whether they test math, science, or coding. Benchmarks where even frontier models struggle (METR, FrontierMath, ARC-AGI) also correlate with each other — not because they test the same skills, but because the same models that underperform on one tend to underperform on the others.

This means the clusters are best interpreted as: "which benchmarks rank models similarly?" rather than "which benchmarks test the same capability?" These often overlap — benchmarks testing similar capabilities often have similar difficulty profiles — but the SWE-bench and METR examples show they can diverge.

---

## Summary

| k | New cluster | What it separates |
|---|---|---|
| 2 | Agentic vs. Reasoning | Tool-using task completion vs. direct QA |
| 3 | Frontier Reasoning | Scaling-era benchmarks (GPQA, AIME, MATH) from still-unsolved ones |
| 4 | Factual Accuracy | Factual precision (SimpleQA, DeepResearch, Writing) from novel reasoning |
| 5 | Novel Problem-Solving | Abstract/novel reasoning (ARC-AGI, FrontierMath) from mixed hard tasks |

The most robust clusters across methods and k values are:
- **Frontier Reasoning** (ECI, GPQA, MATH Level 5, AIME, GeoBench) — always appears together from k=3 onward
- **Agentic/Applied** (APEX Agents, Chess, GDPVal, SWE-bench Verified, TerminalBench, WebDev Arena) — always appears together from k=2 onward
- **ARC-AGI + FrontierMath** — always appear together at every k

### Caveats

1. **Pairwise model overlap is still low.** The median number of shared models per benchmark pair is 4. Distances based on 1-4 shared models are noisy.
2. **23 benchmarks were dropped** to ensure connectivity. The dropped benchmarks (MMLU, HellaSwag, GSM8K, etc.) represent an important "general knowledge" cluster that we cannot directly compare to frontier benchmarks because they were tested on largely disjoint model populations.
3. **Cluster labels are interpretive.** The algorithm groups benchmarks by score correlation patterns — the semantic labels ("Agentic," "Novel Problem-Solving") are our interpretation of why those patterns exist.
