# ML System Design — Simple Guide for Interviews


## Intro — what this is about

We wrote this guide to help machine learning (ML) engineers and data scientists do well in ML system design interviews. It also helps anyone who wants a clear, high-level view of how ML is used in real products.

Many people think ML is just models (like logistic regression or neural networks). But a production ML system is much more than the model. It includes data pipelines, serving infrastructure, evaluation systems, and monitoring so the model keeps working well over time.

Figure 1.1 — high-level ML system 

```
+---------------------------------------------------------------+
|                         ML System                             |
|                                                               |
|  [Data Collection] -> [Data Verification] -> [Feature Extract]|
|                             |                                 |
|                             v                                 |
|                        [ML Algorithm] <-> [Evaluation Pipe]   |
|                           |      ^      [Process Mgmt Tools]   |
|                           v      |                            |
|                     [Analysis Tools]  [Machine Resource Mgmt] |
|                               \     /                         |
|                                [Service Infrastructure]       |
|                                       ^                       |
|                                    [Configuration]           |
+---------------------------------------------------------------+
```

This diagram shows the main pieces: data collection, verification, feature extraction, ML algorithm, evaluation, process management, analysis tools, machine resource management, service infrastructure, and configuration.

## How interviewers judge you

In ML system design interviews you’ll get open questions (for example: design a movie recommender). There’s no single correct answer. Interviewers check:

* your **thought process**
* your knowledge of ML topics
* how you design an end-to-end system
* how you choose trade-offs

Use a framework to structure your answer. Unstructured talk gets confusing.

## A simple framework (use this in interviews)

Follow these steps. Be flexible — if the interviewer wants to focus on one part, follow them.

1. Clarify requirements
2. Frame the problem as an ML task
3. Data preparation
4. Model development
5. Evaluation
6. Deployment and serving
7. Monitoring and infrastructure

Figure 1.2 — steps flow 

```
[Clarify Requirements] -> [Frame ML Task] -> [Data Preparation]
                -> [Model Development] -> [Evaluation]
                -> [Deployment & Serving] -> [Monitoring & Infra]
```

## Step 1 — Clarify requirements

Interview prompts are usually vague. Ask clear questions to set scope. Things to ask:

* Business objective: What does success look like? (e.g., more bookings, more watch time)
* Features to support: What interactions are present? (likes, dislikes, comments)
* Data: What data do we have? Labeled or unlabeled? Size?
* Constraints: CPU/GPU limits? Cloud or device? Privacy rules?
* Scale: Number of users and items? Growth rate?
* Performance: Latency needs vs accuracy?

Write the answers down so interviewer and you agree on scope.

## Step 2 — Frame the problem as an ML task

First ask: does this need ML? In interviews assume ML helps. Then:

* **Define ML objective** (translate business goal to an ML target)

  * Example: business wants more watch time → ML objective = maximize watch time (predict probability a user will watch).
* **Specify inputs & outputs** (what the model sees and returns)

  * Example: harmful content detection: input = post (text + image), output = probability harmful.
* **Choose ML category**: supervised, unsupervised, or reinforcement learning.

  * Most production problems → supervised learning.
* **Pick problem type**: classification, regression, ranking, etc.

Examples (simple mapping)

* Ticket app → objective: maximize registrations
* Video app → objective: maximize watch time
* Ad system → objective: maximize clicks (CTR)

Figure 1.3 — input-output 

```
[User Post (text + image)] -> [Harmful Content Model] -> [Probability of harm]
```

## Step 3 — Data preparation (data + feature engineering)

Good data is essential. Two parts:

A. **Data engineering** — collect, store, ETL

* Data sources: user events, logs, images, third-party APIs.
* Storage: SQL (relational), NoSQL (key/value, document, graph), data lakes.
* ETL: Extract → Transform → Load.

B. **Feature engineering** — pick and process predictive features

* Handle missing values (delete rows/columns or impute mean/median).
* Scale features: normalization (min-max) or standardization (z-score).
* Transform skewed features: log scaling.
* Discretize continuous values (bucket ages/heights).
* Encode categorical features:

  * Integer encoding (when order matters)
  * One-hot encoding (small number of categories)
  * Embeddings (large number of categories)

Figure 1.6 — data pipeline 

```
[Raw Data Sources] -> [Data Engineering] -> [Feature Engineering] -> [Prepared Features Store]
```

Quick data types

* **Structured:** tables (dates, numbers)
* **Unstructured:** text, image, audio, video

## Step 4 — Model development

A. **Model selection**

* Start with a **baseline**: even “recommend most popular” is fine.
* Try simple fast models (logistic regression).
* Move to complex models only if needed (neural nets).
* Consider ensembles (bagging, boosting, stacking) for better accuracy.

Common choices: logistic regression, decision trees, gradient boosting, SVM, factorization machines, deep networks.

Consider: training data needs, training speed, tuning, compute cost, interpretability.

B. **Model training**

* **Construct dataset**: collect raw data, label data, sample, split into train/val/test, address class imbalance.
* **Labels:** hand-labeled (accurate, slow, costly) or natural labels (derived from user actions, cheap).
* **Sampling strategies**: stratified, reservoir, importance sampling, etc.
* **Handle class imbalance**: oversample minority, undersample majority, or use weighted/focal losses.
* **Loss function**: pick based on task (cross-entropy for classification, MSE for regression).
* **Training choices**: scratch vs fine-tuning; distributed training if big models/datasets.

Figure 1.15 — dataset steps 

```
[Collect raw data] -> [Identify features & labels] -> [Sampling strategy]
 -> [Split data] -> [Handle class imbalance]
```

## Step 5 — Evaluation

Two types:

A. **Offline evaluation** (during development)

* Classification: precision, recall, F1, ROC-AUC.
* Regression: MSE, MAE.
* Ranking: Precision@k, nDCG, MRR.
* Generation/NLP: BLEU, ROUGE, FID, etc.

B. **Online evaluation** (production, tied to business)

* Examples: click-through rate, watch time, revenue lift, number of accepted friend requests.

Pick metrics that link to the business objective and explain trade-offs. Also check fairness and bias.

## Step 6 — Deployment & serving

Decide cloud vs on-device.

**Cloud**

* Easier to deploy and update.
* More compute power.
* Higher cost, some latency, less privacy.

**On-device**

* Low latency, better privacy, no network dependency.
* Harder to deploy, limited memory/CPU/battery.

**Model compression** (for on-device):

* Knowledge distillation (train small model to copy a big one).
* Pruning (remove less useful weights).
* Quantization (use fewer bits).

**Testing in production**

* Shadow deployment: run new model alongside old one but don’t serve its results.
* A/B testing: route a subset of traffic to the new model and compare metrics.
* Canary releases, interleaving, multi-armed bandits are other options.

**Prediction pipeline**

* **Batch prediction**: precompute predictions (good when answers can be precomputed).
* **Online prediction**: compute on request (needed when input is dynamic).

Figure 1.21/1.22 — batch vs online 

```
Batch:
[Feature Computation] -> [Precomputed Predictions DB] -> [Client Request -> return precomputed]

Online:
[Client Request] -> [Fetch features] -> [Model] -> [Return prediction]
```

## Step 7 — Monitoring

Monitor both system health and ML quality.

**Operation metrics**: latency, throughput, CPU/GPU usage, error rates.

**ML-specific metrics**:

* Input/output distributions and drift detection.
* Model accuracy on sampled labeled data.
* Model version and rollout health.

Common problem: **data distribution shift** — data in production changes from training data. Fix by retraining regularly or use larger training sets.

Figure 1.24 — data shift 

```
[Training Data Distribution] != [Serving Data Distribution]
 -> Model performance drops
 -> Retrain / monitor / adapt
```

