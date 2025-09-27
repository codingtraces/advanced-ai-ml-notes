# Machine Learning System Design: A Practical Framework for Interviews and Production

*Helping ML engineers and data scientists think end-to-end — from problem to production.*

---

## Why this matters

Machine learning is more than picking an algorithm. In production, ML systems are software ecosystems: data ingestion, feature stores, training pipelines, serving infrastructure, evaluation, monitoring, and continual improvement all matter. This post condenses a practical, interview-ready framework for designing ML systems — the same framework you can use when asked to design a recommendation engine, content filter, or search system.

---

# Figure 1.1 — Components of a production-ready ML system (ASCII)

```
+---------------------------------------------------------------+
|                         ML System (box)                       |
|                                                               |
|  [Data Collection] -> [Data Verification] -> [Feature Extract]|
|                                                               |
|                           +-------------+                     |
|                           |  ML Agent   |  <--- Evaluation     |
|                           | (ML Alg.)   |       Pipeline      |
|                           +-------------+                     |
|                         /      |      \                       |
|    [Process Mgmt] ---- /       |       \ ---- [Analysis Tools] |
|                        /        v        \                    |
|               [Machine Resource Mgmt]   [Service Infrastructure]
|                                                               |
|                           [Configuration]                     |
+---------------------------------------------------------------+
```

> Short explanation: data flows from left → ML core → evaluation & analysis on the right; process management, resource mgmt and configuration sit around the system.

---

# Figure 1.2 — ML system design steps (ASCII flow)

```
Clarify requirements
        |
        v
Frame as ML task
        |
        v
  Data preparation
        |
        v
  Model development
        |
        v
    Evaluation
        |
        v
Deployment & Serving
        |
        v
 Monitoring & Infrastructure
```

Use this sequence as your backbone in interviews. Be flexible: if the interviewer cares most about model architecture, focus there — but show awareness of all stages.

---

## Step 1 — Clarify requirements

Interview prompts are intentionally vague. Ask focused questions and write down the agreed scope:

* **Business objective** — what metric matters? (e.g., bookings, revenue, retention)
* **Features** — what user interactions exist (likes, clicks, ratings)?
* **Data** — sources, size, label availability?
* **Constraints** — compute budget, cloud vs on-device, privacy constraints
* **Scale** — users, items, growth rates
* **Performance** — latency vs accuracy trade-offs

Writing requirements down demonstrates structure and alignment.

---

# Figure 1.3 — Harmful content detection: input → system → probability (ASCII)

```
   +---------+
   |  Post   |  (text + image + metadata)
   +---------+
        |
        v
+---------------------------+
| Harmful content detection |
|        (ML system)        |
+---------------------------+
        |
        v
+---------------------------+
|       Probability p       |
|   (likelihood of harmful) |
+---------------------------+
```

Notes: Might be multiple models (violence model, nudity model) whose outputs combine into a final decision.

---

## Step 2 — Frame the problem as an ML task

Translate business goals into a measurable ML objective:

* Define ML objective (maximize CTR, minimize harmful-post false positive rate, maximize watch time).
* Specify inputs and outputs (single model vs multiple submodels).
* Choose ML category: supervised / unsupervised / reinforcement. Most production tasks are **supervised**.

Two ways to structure model IO (scenario examples):

```
Scenario 1: Per-user probabilities for all items
[User A] --> [Model] --> [p1, p2, p3, ... pN]

Scenario 2: Per (user, item) probability
[User A] + [Event 4 features] --> [Model] --> [p]
```

Pick the framing that best fits data availability, latency and product needs.

---

# Figure 1.5 — ML categories (mini ASCII tree)

```
Machine Learning
├─ Supervised
│  ├─ Classification
│  │  ├─ Binary
│  │  └─ Multiclass
│  └─ Regression
├─ Unsupervised
│  ├─ Clustering
│  ├─ Association
│  └─ Dimensionality reduction
└─ Reinforcement Learning
```

---

## Step 3 — Data preparation

ML models learn from data. Two essential practices: **data engineering** and **feature engineering**.

### Data engineering basics

* **Data sources**: who collected it, how clean, trustworthiness
* **Data storage**: SQL (Postgres/MySQL) vs NoSQL (Redis, DynamoDB), data warehouses/lakes
* **ETL**: Extract → Transform → Load

# Figure 1.8 — ETL (ASCII)

```
[Database]  [Logs]  [Flat files]
    \         |        /
     \        |       /
      \       v      /
   +---------------------+
   |      TRANSFORM      |   <- cleansing, mappings, joins
   +---------------------+
            |
            v
+----------------+  +----------------+  +----------------+
| Data Warehouse |  |    Database    |  |   Flat files   |
+----------------+  +----------------+  +----------------+
```

### Data types (structured vs unstructured)

* **Structured**: numerical (continuous/discrete), categorical (nominal/ordinal)
* **Unstructured**: text, images, audio, video

Understanding the data type helps pick models: classical ML for structured data, deep learning for many unstructured tasks.

---

## Feature engineering

Two steps:

1. Use domain knowledge to select predictive features.
2. Transform features into model-ready form.

Common operations:

* Missing values: deletion or imputation (mean/median/default)
* Scaling: normalization (min-max), standardization (z-score), log-scaling
* Discretization: bucketing continuous values
* Categorical encoding: integer encoding, one-hot encoding, embeddings

# ASCII example — One-hot vs Integer vs Embedding

```
Color column: [Red, Green, Blue]

Integer encoding:
 Red -> 1
 Green -> 2
 Blue -> 3

One-hot encoding:
 Red  -> [1,0,0]
 Green-> [0,1,0]
 Blue -> [0,0,1]

Embedding (learned vector):
 Red  -> [0.12, -0.3, 0.6, ...]
```

Embeddings are preferred for very high-cardinality categories.

---

## Step 4 — Model development

### Model selection workflow

1. Baseline (popularity, simple heuristics)
2. Simple models (logistic / linear)
3. Complex models (GBDT, neural networks)
4. Ensembles (bagging, boosting, stacking)

Mention trade-offs: data needed, training time, inference latency, interpretability, and compute (GPU vs CPU).

### Model training: dataset construction

Steps:

1. Collect raw data
2. Identify features & labels (hand-labeled vs natural labels)
3. Choose sampling strategy (stratified, reservoir, etc.)
4. Split data (train / val / test)
5. Handle class imbalance (oversample / undersample / weighted loss)

# Figure 1.15 — Dataset construction (ASCII)

```
[Collect raw data]
        |
   [Identify features & labels]
        |
 [Select sampling strategy]
        |
     [Split data]
        |
[Address class imbalance if any]
```

Labeling strategies:

* Hand-labeling: accurate but costly and slow
* Natural labels: infer from user actions (like=1, not-like=0)

Handling imbalance: oversampling minority, undersampling majority, or weighted/focal losses.

---

## Step 5 — Evaluation

Two kinds of evaluation:

### Offline metrics

* Classification: precision, recall, F1, ROC-AUC, PR-AUC
* Regression: MSE, MAE, RMSE
* Ranking: Precision@k, MRR, nDCG
* NLP / generation: BLEU, ROUGE, CIDEr; images: FID

### Online metrics (business-aligned)

* CTR, watch time, revenue lift, prevalence/appeals for content-safety systems

Interview tip: choose metrics that map clearly to the business objective and explain trade-offs.

---

## Step 6 — Deployment & serving

Topics to discuss: cloud vs on-device, model compression, testing strategies, prediction pipeline (batch vs online).

### Cloud vs On-device (summary)

```
Cloud:
 + easier to manage
 - cost, privacy, network latency

On-device:
 + low-latency, privacy
 - hardware limits, complexity of deployment
```

### Model compression

* Knowledge distillation
* Pruning (sparsity)
* Quantization (fewer bits per parameter)

### Testing in production

* Shadow deployment (run new model; don't serve its outputs)
* A/B testing (random traffic split; ensure statistical power)
* Canary releases, interleaving experiments, bandit methods

# Figure — Shadow deployment (ASCII)

```
All requests ->  +----------------------------+
                |  In-production ML system   | -> responses to users
                +----------------------------+
                       ^
                       |  (also)
                       |
                  +------------------+
                  |  New ML system   | -> store predictions for analysis
                  +------------------+
```

### Prediction pipeline: batch vs online

* **Batch** — precompute predictions periodically (good for large-scale, non-real-time)
* **Online** — compute on demand (needed for real-time personalization or interactive apps)

---

# Figure 1.23 — Example: Personalized News Feed (ASCII simplified)

```
[User Request]
     |
     v
[Retrieval Service]  <-- fetch candidate posts
     |
     v
[Ranking Service] <- (Online features) + (Feature store precomputed features)
     |
     v
[Re-ranking service / Re-ranker]
     |
     v
[Top K posts] -> [User]
```

Data-prep pipeline:

```
Stream of interactions -> Batch feature computation -> Feature store
```

---

## Step 7 — Monitoring

Monitoring is vital to detect failures and drift.

### Why systems fail

* **Data distribution shift**: model trained on distribution A, serving sees distribution B
* Stale models as the world changes

### What to monitor

* **Operational metrics**: latency, throughput, errors, CPU/GPU utilization
* **ML metrics**: input/output distributions, drift detectors, model accuracy (when labels arrive), model version usage

Remedies:

* Train on diverse/large data
* Retrain regularly or trigger retraining on drift
* Version models and keep rollback plans

---

## Quick checklist to use in an interview

1. Clarify product goals & constraints — write them.
2. Frame the ML task and define inputs/outputs.
3. List available data & storage plan.
4. Sketch ETL → Feature Store → Model pipeline.
5. Propose baseline model + escalation plan.
6. Choose offline & online metrics mapping to product.
7. Suggest deployment plan (cloud/on-device, batch/online).
8. Describe testing strategy (shadow/A-B) & rollout plan.
9. Define monitoring and retraining cadence.

---

## Summary

A strong ML system design answer is structured, pragmatic, and mindful of trade-offs across data, modeling, evaluation, deployment, and monitoring. Use the seven-step framework above to guide your responses. Show alignment with business goals, awareness of data realities, model limitations, and operational complexity — that’s what interviewers look for.

---

If you want I can:

* Convert this to a **one-page PDF** or **GitHub README** file.
* Produce a **one-page cheat-sheet** for interview prep.
* Generate **example answers** applying this framework to specific prompts (e.g., design a movie recommender or harmful-content detector), with ASCII diagrams included.

Which would you like next?
