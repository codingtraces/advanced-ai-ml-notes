# Visual Search System — Simple Guide

This is a simple blog-style explanation of how to design a **visual search system** (like Pinterest visual search). 

## What the system does

A visual search system helps users find images that look like a chosen image. The user can select an image or crop a part of an image (e.g., a chair). The system finds and ranks images that are visually similar and shows them to the user.

Quick visual (Figure 2.1)

```
+-----------------------------------------------------+
| [Big Query Image]    Filters: eames | chair | wood   |
|                                                     |
| -> Similar results:                                 |
|  [img1][img2][img3][img4]  (card: desc + source)    |
|  [img5][img6][img7][img8]                            |
+-----------------------------------------------------+
```

## Clarify requirements (example Q&A)

When designing, first ask questions and agree on scope. Example conversation:

* Q: Should results be ranked by similarity?
  A: Yes — most similar first.

* Q: Support videos?
  A: No — only images.

* Q: Support cropping a part of an image?
  A: Yes.

* Q: Personalized results per user?
  A: No — same results for every user (simpler).

* Q: Use image metadata (tags) to search?
  A: For simplicity: no — use pixels only.

* Q: Which user actions are available to collect labels?
  A: Only clicks (we’ll treat clicks as weak signals).

* Q: Moderate content (e.g., block NSFW)?
  A: Important but out of scope for basic design.

* Q: How fast and how big?
  A: Must handle retrieval quickly even at 100–200 billion images.

## Problem summary

* **Input:** query image (or crop).
* **Output:** ranked list of visually similar images.
* **Assumptions:** images only, no personalization, only pixel-based similarity, clicks available as weak labels, must scale to billions of images.

## Frame the problem as an ML task

* This is a **ranking** problem: show the most similar images first.
* Use **representation learning**: map images to vectors (embeddings).

  * Similar images → nearby vectors in embedding space.
* Search = compute query embedding → find nearest vectors in index → return ranked images.

Simple flow (Figures 2.2 & 2.4)

```
[Query image] -> [Embedding model] -> E_q (vector)
Find nearest E_* to E_q in index -> [Ranked similar images]
```

## Data available

Main data types:

1. **Images** — raw files + metadata (owner_id, upload time, tags).
   Example:

   ```
   Images:
   ID | OwnerID | UploadTime | ManualTags
   1  | 8       | 1658451341 | Zebra
   ```

2. **Users** — basic user info (not used for personalization here).

   ```
   Users:
   ID | Username | Age | City
   1  | johnduo  | 26  | San Jose
   ```

3. **User-image interactions** — impressions, clicks, position, timestamp.

   ```
   Interactions:
   UserID | QueryImageID | DisplayedImageID | Pos | Type  | Timestamp
   8      | 2            | 6                | 1   | Click | 1658450539
   ```

## Feature engineering for images

Images must be preprocessed before feeding the model:

* Resize to a fixed size (e.g., 224×224).
* Scale pixels to [0,1] or z-score normalize.
* Make sure color mode is consistent (RGB).
* Apply data augmentation during training (random crop, flip, color jitter).

## Model development

Model choice

* Use **neural networks** to generate embeddings from images.
* Good architectures: **ResNet** (CNN) or **Vision Transformer (ViT)**.
* Output is a fixed-size vector (embedding) per image.

Simple architecture (Figure 2.5)

```
[Input 224x224] -> Conv / Transformer layers -> Fully connected -> Embedding (size N)
```

How to train embeddings

* Use **contrastive learning**: teach the model to bring similar images closer and push dissimilar images away in embedding space.
* Training sample: (query q, positive p, n−1 negatives). The label indicates which of the n items is the positive.

Ways to get positives for training

1. **Human labeling** — accurate but expensive and slow.
2. **User clicks** — automatic but noisy and sparse.
3. **Self-supervision / augmentation** — create positives by augmenting q (rotate, crop, color change). Works well at scale (SimCLR, MoCo).

Recommended starting point: **self-supervised (augmentation)** — no labeling cost and scales to billions of images. Later, combine with click data or human labels if needed.

Loss function (high-level)

* Compute embeddings E_q, E_p, E_neg.
* Measure similarity (dot product or cosine).
* Apply softmax over similarities and use cross-entropy so the model learns to give high score to the positive and low scores to negatives. This is a simple contrastive loss view.

## Evaluation

Offline metrics

* **nDCG (Normalized Discounted Cumulative Gain)**: good for graded relevance (scores 0..5). Measures ranking quality and gives more weight to top positions. Use this when ground-truth similarity scores are available.
* **mAP (mean Average Precision)**: good for binary relevance (relevant / not relevant).
* **Precision@k**, **Recall@k**, **MRR** exist but each has limits; nDCG and mAP are often better for ranking quality.

Online metrics

* **Click-through rate (CTR)** on results (how often users click suggestions).
* **Time spent** viewing suggested images (engagement).
* Business metrics: saves, purchases, conversions if integrated.

## Serving: pipelines & components

Two main pipelines:

1. **Indexing pipeline** (offline / background)

   * New images arrive → compute embeddings → add embedding to index table.
   * Keep index updated (ingest new images, remove deletions).
   * Compress embeddings (vector/product quantization) to save memory if needed.

2. **Prediction pipeline** (online)

   * Preprocess query image → generate embedding via model → nearest neighbor search over index → re-rank results with business logic → return results.

ASCII: end-to-end (Figure 2.19)

```
Indexing:
[New images] -> [Store raw] -> [Indexing service computes embeddings] -> [Index table]

Prediction:
[Query image] -> [Preprocess] -> [Embedding service (model)] -> [ANN search] -> [Re-rank] -> [Return results]
```

## Nearest neighbor search (NN)

Goal: find k nearest embeddings to the query embedding.

* **Exact NN (linear search)**: compute distance to every embedding (O(N×D)). Too slow at billions of images.
* **Approximate NN (ANN)**: much faster, returns “close enough” neighbors. Common and practical.

ANN categories:

* **Tree-based**: partition space into regions (KD-tree, R-tree, Annoy).
* **LSH (Locality Sensitive Hashing)**: hash vectors so similar ones fall into same buckets.
* **Clustering-based**: cluster the dataset and search within nearby clusters.

Common tools: **Faiss** (Meta), **ScaNN** (Google). These provide optimized ANN implementations.

ASCII: ANN idea (LSH buckets) (Figure 2.24)

```
[Embeddings] --LSH--> [Bucket 42]  (search only bucket(s) for query)
```

Index storage & optimizations

* Use **product quantization** or **vector quantization** to compress embeddings and reduce memory.
* Trade-off: more compression → less memory but lower search precision.
* Keep balance between memory, latency, and precision.

## Re-ranking & business rules

After ANN returns candidates, apply post-processing:

* Remove private or deleted images.
* Remove exact duplicates or near-duplicates.
* Filter unsafe or disallowed content.
* Optionally use metadata (tags, popularity) to adjust final ranking if allowed.

## Real-world mini-example: chair search

User crops a chair from a photo:

1. Crop → preprocess to 224×224.
2. Compute embedding via model (ResNet).
3. Query ANN index for top-100 candidates.
4. Re-rank results: remove duplicates, boost in-stock items, apply color/style filters.
5. Return top 10 images with source and price.

## Performance & trade-offs

* **ANN** speeds up search but may not return the exact nearest neighbors. For UX, “close enough” is usually fine.
* **Embedding quality** matters: better trained embeddings → better search results.
* **Index freshness vs cost**: frequently re-indexing keeps results fresh but costs compute.
* **Storage vs latency**: precompute and store more to reduce latency, at the cost of memory.

## Advanced topics (brief)

* **Content moderation** to block inappropriate images.
* **Biases** (positional bias, presentation bias) and how to measure/mitigate.
* **Use metadata (tags, text)** to improve results with multimodal models.
* **Smart crop** using object detection to make better query crops.
* **Graph neural networks** to model relations between images.
* **Active learning / human-in-the-loop** for labeling hard cases.

## Short checklist for interviews

1. Clarify scope (crop, personalization, use of metadata, scale).
2. State ML objective: rank visually similar images.
3. Describe data: images, interactions, metadata.
4. Model: produce embeddings (ResNet/ViT), train with contrastive/self-supervised loss.
5. Index: ANN (Faiss/ScaNN), use quantization to save memory.
6. Serving: embedding service → ANN → re-rank → return.
7. Evaluation: offline (nDCG, mAP), online (CTR, engagement).
8. Monitoring: search quality, latency, index freshness.
9. Mention trade-offs: accuracy vs latency, storage vs throughput.
