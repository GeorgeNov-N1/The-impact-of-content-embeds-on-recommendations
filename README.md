# The-impact-of-content-embeds-on-recommendations
# Hybrid Recommendation Systems: Impact of Content Embeddings

## Overview

Modern recommender systems often combine collaborative filtering (CF) with content-based signals to improve recommendation quality. While content embeddings are widely used to address cold-start and sparsity issues, their actual impact on ranking quality is highly scenario-dependent and often less stable than commonly assumed.

This project investigates how different types of content embeddings (textual, visual, and multimodal) affect recommendation quality across multiple realistic scenarios, including active users, sparse users, and cold-start items. In addition to average offline performance, we explicitly analyze variance across random seeds, signal quality effects, and failure modes of hybrid recommendation systems.

The goal of this project is not to optimize a single metric, but to understand when content embeddings help, when they hurt, and why.

---

## Dataset

We use the **VK-LSVD** open dataset, which contains:
- user–item interaction logs,
- item metadata,
- precomputed content embeddings.

Dataset source:  
https://huggingface.co/datasets/deepvk/VK-LSVD

To ensure that rare but practically important scenarios (e.g. cold-start items) are observable, experiments are conducted both on the full dataset and on carefully constructed subsamples.

---

## Problem Setting

Given a set of users \( U \), items \( I \), and interaction history \( R \), the task is to generate a ranked list of items for each user.

We compare:
- **CF-only models**, relying exclusively on interaction data,
- **Hybrid models**, which incorporate item-level content embeddings.

Crucially, the analysis is performed not only on the full dataset, but on targeted user and item segments that are known to stress recommender systems in production.

---

## User and Item Segmentation

To reflect realistic deployment conditions, we evaluate models under the following scenarios.

### User Groups
- **Sparse users** — users with fewer than \( K \) interactions in the training set.
- **Active users** — users with at least \( K \) interactions in the training set.

### Item Groups
- **Cold-start items** — items with at most \( N \) interactions in the training set.
- **Popular items** — items above the cold-start threshold.

Thresholds \( K \) and \( N \) are fixed per experiment and varied to test robustness.

---

## Models

We evaluate a minimal but representative set of models to isolate the effect of content signals:

| ID | Model |
|----|------|
| M0 | CF-only baseline |
| M1 | CF + text embeddings |
| M2 | CF + image embeddings |
| M3 | CF + multimodal embeddings |

All models share the same optimization procedure and evaluation protocol.  
The only difference is the presence and type of content features.

---

## Evaluation Protocol

### Data Splitting

We use a user-level **leave-last-out** strategy:
- the most recent interaction(s) per user are held out for validation,
- the remaining interactions are used for training.

This setup approximates a realistic recommendation scenario while preserving temporal causality.

### Metrics

We report:
- **NDCG@10** — ranking quality,
- **Recall@10** — coverage of relevant items.

Metrics are computed only on the target user/item subsets relevant to each hypothesis.

---

## Research Hypotheses

### H1 — Active Users (Risk of Content Noise)

For active users with rich interaction histories, adding content embeddings does not improve recommendation quality compared to CF-only models and may degrade ranking performance due to noisy or conflicting signals.

**Motivation:**  
Collaborative filtering already provides a strong preference signal for active users. Additional content features may interfere with fine-grained preference modeling.

---

### H2 — Sparse Users (Benefit with Fragility)

For sparse users, content embeddings can improve recommendation quality compared to CF-only models; however, this improvement is fragile and highly sensitive to the quality of both interaction signals and content representations.

**Motivation:**  
When collaborative signals are weak, content features provide useful inductive bias, but errors in embeddings or weak feedback can easily dominate learning.

---

### H3 — Cold-Start Items (Necessity of Content, High Variance)

For cold-start items, content embeddings are necessary for achieving non-zero recommendation quality, as CF-only models consistently fail in the absence of interaction data.

However, the benefit of content embeddings:
- exhibits high variance across random seeds,
- strongly depends on the type of positive feedback signal.

In particular, explicit feedback signals (likes, shares, bookmarks) amplify the effect of content embeddings by up to **4×** compared to implicit consumption signals.

**Motivation:**  
For cold-start items, content representations act as the only bridge to the recommendation space. Their effectiveness depends on how well sparse user feedback anchors these representations.

---

## Experimental Design

Each hypothesis is evaluated using the same experimental pipeline, applied to different data slices.

### Scenario-Based Evaluation

| Scenario | Users | Items | Purpose |
|--------|------|------|--------|
| Active users | Active | All | Test content interference |
| Sparse users | Sparse | All | Test benefit and sensitivity |
| Cold-start items | All | Cold-start | Test necessity and stability |

To reduce false conclusions driven by randomness, all experiments are repeated across multiple random seeds.

---

### Robustness Experiments

To assess stability, we introduce controlled noise into content embeddings:
- Gaussian noise with increasing variance,
- Random feature dropout.

We analyze how recommendation quality degrades under these perturbations, highlighting failure modes of hybrid models.

---

## Key Findings (High-Level)

- CF-only models completely fail on cold-start items, yielding zero ranking quality.
- Content embeddings are necessary but not sufficient for cold-start recommendation.
- The effectiveness of content embeddings is:
  - unstable under extreme sparsity,
  - strongly amplified by explicit user feedback.
- Hybrid models may hurt performance for active users, confirming that content signals are not universally beneficial.

---

## Limitations and Future Work

- Offline metrics may not fully capture online user behavior.
- Extremely sparse validation sets introduce high variance.
- Exploration–exploitation dynamics are not modeled.

Future work may include:
- online evaluation or counterfactual estimation,
- adaptive weighting of content signals,
- confidence-aware or uncertainty-aware hybrid models.

---

## Repository Structure (Planned)
