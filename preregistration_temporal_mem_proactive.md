# Pre-Registration: Controlled Degradation Experiments on Memory Consolidation Quality for Proactive Agents

**Date**: 2026-04-17  
**Project**: temporal_mem_proactive  
**Target Venue**: NeurIPS 2026 Evaluations & Datasets Track (deadline: May 6, 2026)  
**Status**: Pre-registered before data collection

## Research Question

Does memory consolidation quality causally affect proactive agent performance? Which quality dimensions (temporal coverage, semantic coherence, compression fidelity, temporal ordering) most strongly influence proactive demand detection on LatentNeeds-Bench?

## Pre-Registered Hypotheses

### H1: Semantic Coherence Dominates Temporal Ordering

**Hypothesis**: Semantic coherence degradation will impact proactive performance more than temporal ordering degradation (both SimpleMem and Mem0 systems).

**Rationale**: Latent need detection requires thematic clustering of user behavior patterns rather than chronological accuracy.

**Statistical Test**: Bootstrap comparison of linear slopes of the two degradation curves. One-sided test. H1 supported only if both systems show consistent direction.

**Success Criterion**: Both systems show semantic coherence slope > temporal ordering slope; one-sided bootstrap p < 0.05.

**Result interpretation**:
- Both systems show semantic_slope > temporal_slope (p < 0.05) → H1 CONFIRMED
- One system significant, one not → H1 PARTIAL (exploratory; analyze possible explanations for divergence)
- Neither system significant → H1 NULL

### H2: Compression Fidelity Shows Non-Linear (Concave) Degradation

**Hypothesis**: Compression fidelity follows a concave degradation curve: stable until >50% degradation, then sharp drop.

**Rationale**: Consolidated summaries need only capture key behavioral patterns. Below ~50% information density, patterns become unrecoverable.

**Statistical Test**: Session-level Wilcoxon signed-rank test, one-sided (H0: Δ ≥ 0), where Δ = F1(50%) - [F1(0%) + F1(100%)]/2 for each session (n=100 sessions).

**Success Criterion**: F1 at 50% < linear interpolation; bootstrap 95% CI excludes zero.

### H3: Session Length Moderates Temporal Coverage Effect

**Hypothesis**: Temporal coverage degradation has stronger impact on sessions >30 turns than <20 turns.

**Prerequisite**: ≥30 sessions with <20 turns AND ≥30 sessions with >30 turns in LatentNeeds-Bench. If prerequisite fails, H3 downgraded to exploratory.

**Rationale**: Shorter sessions fit in LLM context windows without consolidation; longer sessions require consolidation.

**Statistical Test**: Bootstrap interaction test comparing temporal coverage slopes between session-length groups. Two-sided.

**Success Criterion**: Bootstrap 95% CI of (slope_long - slope_short) excludes zero in predicted direction.

### H4: Non-Linear Degradation Curves (Exploratory)

**Hypothesis**: Each degradation dimension shows non-linear deviation from linear interpolation.

**Constraint**: We do NOT claim to identify functional form (power/sigmoid/piecewise) — 5 data points are insufficient. Only direction of non-linearity is reported.

**Statistical Test**: Wilcoxon signed-rank test on deviations of 25%/50%/75% points from linear interpolation. Bonferroni correction applied to 2 confirmatory dimensions only (semantic coherence + compression fidelity): α = 0.05/2 = 0.025. Temporal ordering and temporal coverage treated as exploratory (uncorrected α = 0.05). Exploratory.

**Success Criterion**: ≥2 of 4 dimensions show significant non-linear deviation (exploratory only).

## Experimental Design

### Memory Systems
- **Primary**: SimpleMem + Mem0 (fact extraction consolidation, local vLLM endpoint via mem0ai) — both full degradation curves, 4 dims × 5 levels
  - Note: TiMem was originally planned as second system but excluded because it requires a cloud-only API key incompatible with our controlled-backbone design requirement (all systems must use the same local vLLM endpoint).
- **Baseline**: RAG-only anchor (0% degradation)
- **Dimensional Atlas**: 1 additional system (natural CQS profile, no degradation)

### Benchmark
- **Primary**: LatentNeeds-Bench (PASK, github.com/xzf-thu/Pask) — 100 sessions, 3,936 turns, CC BY-NC-SA 4.0
- **Cross-validation**: TBD (ProactiveBench excluded due to insufficient session count (12 sessions); PARE excluded due to no public data release; LatentNeeds-Bench is primary benchmark)

### Degradation Protocol
- **Dimensions (4)**: (a) Temporal Coverage — random entry dropping; (b) Temporal Ordering — timestamp shuffling; (c) Semantic Coherence — cross-topic entity replacement; (d) Compression Fidelity — replace consolidated summaries with truncated raw episodes
- **Severity Levels (5)**: 0%, 25%, 50%, 75%, 100%
- **Total conditions**: 4 × 5 = 20 per system; 100 sessions per condition

### LLM Backbone
- **MVP**: Qwen2.5-7B-Instruct via local vLLM (OpenAI-compatible endpoint)
- **Full Experiment**: Qwen3-32B via local vLLM (tensor parallel, 2× RTX PRO 6000 96GB)
- All memory consolidation routes through the same local LLM (controlled variable)

### Consolidation Quality Score (CQS)
Four sub-metrics computed post-hoc:
- **TCR**: Temporal Coverage Ratio (fraction of timeline covered)
- **SCS**: Semantic Coherence Score (pairwise cosine similarity of same-entity entries)
- **ICR**: Information Compression Rate (BERTScore normalized by token reduction)
- **TOA**: Temporal Ordering Accuracy (Kendall's tau)

CQS = arithmetic mean of four normalized sub-metrics (equal weights).

### Statistical Analysis
- Paired bootstrap CI (n=10,000, seed=42) for all hypothesis tests
- Session-level F1 used for paired bootstrap

## Failure Modes & Pre-Planned Responses

| Failure | Response |
|---------|----------|
| LatentNeeds-Bench insensitive to memory quality | Reframe as null-result study or postpone to ICLR 2027 |
| H3 prerequisite fails | H3 downgraded to exploratory |
| TiMem incompatible with local LLM | Replace with Mem0 or MemReader |
| All effects null | Reframe as null result |

## Timeline

| Date | Milestone |
|------|-----------|
| 2026-04-17 | Pre-registration commit |
| 2026-04-17–19 | MVP validation |
| 2026-04-20–28 | Full degradation experiments |
| 2026-04-28–May 1 | Analysis and figures |
| 2026-May 1–6 | Paper writing and submission |

*Pre-registered before any experimental data collection. Post-registration modifications tracked via git history.*
