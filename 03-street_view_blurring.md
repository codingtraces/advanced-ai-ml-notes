# Street View Blurring — Conceptual Interview Guide (simple, interview-friendly)

**One-line summary:**
Automatically find faces and license plates in street images and blur them to protect privacy.

---

## One-sentence starter you can use in interviews
"We build a detection pipeline that finds faces and plates in street images, blurs them offline, and uses user reports and retraining to improve coverage." 

Say this first to show clarity.

---

## The simple mental model (3 bullets)
- **Detect**: find where faces and plates are in the picture.
- **Blur**: apply a visual filter to hide those regions.
- **Improve**: use user reports and retraining to fix misses.

---

## Short ASCII diagrams to explain quickly

**A) Problem & goal**
```
 Original image ---> [Detect boxes for faces & plates] ---> [Blur boxes] ---> Shown to user
```

**B) High-level system flow**
```
   Data ingestion (cars + cameras)
           |
        Preprocess
           |
     Model training  <---- human labels / hard-negatives
           |
   Batch prediction (offline)  ---> blur service ---> store blurred images
           |
        Serving
           |
     Users + reports ---> data pipeline (improve model)
```

**C) Bounding box issue (NMS idea — visual)**
```
Many overlapping boxes around same face:  [ ] [   ] [  ]
Keep highest-score box and drop overlaps => one neat box per object
```

---

## Key conceptual trade-offs to highlight (one-liners)
- **Accuracy vs cost**: More labeling and bigger models increase accuracy but cost money and time.
- **Offline vs real-time**: Blurring can be done offline for quality; real-time would cost much more.
- **Precision vs recall**: High precision means few false blurs; high recall means few missed faces. For privacy, prioritize recall but avoid over-blurring.
- **Automation vs human checks**: Fully automatic scales; human-in-the-loop fixes edge cases.

---

## Simple answers to common interview questions (memorize these short replies)
- **How do you get training data?**
  "Start with sampled annotated images, expand with augmentation, and collect user reports to add hard examples for retraining." 

- **How to reduce duplicate boxes?**
  "Use Non-Maximum Suppression: keep the best box and remove overlapping ones." 

- **Which errors are worse?**
  "Missing a face (privacy leak) is usually worse than blurring something harmless — so favor recall with controlled precision." 

- **How to measure success?**
  "Offline: AP/mAP (detection accuracy). Online: user report counts, and manual audits for quality." 

---

## Tiny glossary of buzzwords (say 4–6 in interviews)
- **Object detection:** Finding objects and their locations.
- **Bounding box:** The rectangle that marks an object in an image.
- **Non-Maximum Suppression (NMS):** Remove duplicate overlapping boxes.
- **Data augmentation:** Make more training examples by slightly changing images.
- **Hard negative mining:** Add tricky false-positive cases into training so model learns them.
- **Human-in-the-loop:** Use people to check model outputs and correct labels.

Memorize 4 of these and a one-liner each.

---

## Short interview script (five lines to read quickly)
1. One-liner: "We detect faces and plates and blur them to protect privacy."
2. How: "Train a detector on labeled images, run batch predictions to blur images, and handle user reports." 
3. Trade-off: "We prioritize recall to avoid privacy leaks, but moderate precision to avoid excessive blurring." 
4. Metrics: "mAP offline; user reports and audit pass rates online." 
5. Extension: "Add continual retraining using hard negatives from reports and consider GDPR compliance." 

---

## Quick practice prompts (answer in 30–60s)
- "Explain how you'd design a privacy blurring feature for street images." 
- "What would you do if the model keeps missing small license plates?" 
- "How would you keep the system scalable as images grow?"

---

## Final tip
Start answers with the one-liner, follow the five-line script, and always mention a clear trade-off. Interviewers care more about how you reason and choose trade-offs than deep architecture details.

If you'd like, I can convert this into 6 flashcards, make a printable cheat-sheet, or produce 3 mock interview Q&As for the Street View blurring topic.