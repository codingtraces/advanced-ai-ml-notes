# Visual Search

---

## One-sentence answer you can start with
"A visual search system finds pictures that look like a photo you give it by turning images into simple number summaries, then finding other images whose summaries are close to the query's summary."

Say this first in the interview to show clarity.

---

## Big-picture idea (3 steps)
1. **Turn every image into a short summary (embedding).** Think of the summary as a location on a map.
2. **When a user gives a photo, convert it to its location.**
3. **Find other images whose locations are close and show them.**

This is the core mental model — practice explaining it like this.

---

## Simple ASCII diagrams to explain quickly

**A) Embedding idea (map view)**

```
  Embedding space (2D view):

     y2 |       x   x   x      x
        |   x   x   q   x  x
        |       x   x   x      x
     ---+-------------------------
          x1 ->

  q = query image location. Nearby x's = similar images.
```

**B) System flow (very short)**

```
 [User image] -> [Preprocess] -> [Model -> embedding] -> [Nearest-neighbor search] -> [Re-rank & filter] -> [Results]
```

**C) Index vs query (how index helps)**

```
 Indexing (offline):  New images -> summarize -> store summaries in index
 Query (online):      User image -> summarize -> search index -> return close summaries
```

Use these diagrams to point-and-explain in interviews.

---

## Key trade-offs 
- **Accuracy vs speed:** Perfect similarity is slow at scale; "good enough" and fast is usually better.
- **Cheap labels vs good labels:** Auto-labels (augmentations/clicks) are cheap but noisy; human labels are expensive but clean.
- **Memory vs speed:** Storing everything gives accuracy but costs memory; compressing saves memory but loses a bit of accuracy.
- **Simplicity vs complexity:** Start simple (pretrained model + basic search) and add layers (filters, metadata, personalization) when needed.

State one trade-off, then give a 1-sentence solution: e.g., "Prefer ANN + compression when we have billions of images; use exact search for small datasets."

---

## Questions
- **How would you get training labels?**
  "Start with automated positives via slight image changes (cheap). Then gradually add click-based signals and human checks to clean up noisy cases."

- **How to scale search for billions of images?**
  "Don't compare to every image — pre-compute short summaries and use a fast index to search nearby summaries."

- **How to measure success?**
  "Offline: a ranking score that rewards putting best matches at the top (nDCG-style idea). Online: clicks and time spent — but watch for biases."

- **What if results are biased or strange?**
  "Check data collection and position effects. Fix by cleaning labels, adjusting ranking, or adding fairness rules in re-ranking."

---

## Keywords
- **Embedding / representation:** Short numeric summary of an image.
- **Contrastive learning:** Training style that pulls similar items together and pushes others apart.
- **Pretrained model / transfer learning:** Start with a model already trained on lots of images to save time.
- **Approximate nearest neighbor (ANN):** Fast way to find "close" items without checking everything.
- **Product quantization / vector quantization:** Ways to compress summaries to save memory.
- **Re-ranking:** Final cleanup step that enforces business rules and filters bad results.
- **Self-supervised learning:** Learning from the data itself (no labels), often by augmenting images.
- **A/B testing:** Compare two versions of the system live to see which engages users more.
