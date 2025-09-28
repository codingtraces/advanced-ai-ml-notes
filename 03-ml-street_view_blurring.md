# Street View Blurring

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

## Key conceptual trade-offs to highlight 
- **Accuracy vs cost**: More labeling and bigger models increase accuracy but cost money and time.
- **Offline vs real-time**: Blurring can be done offline for quality; real-time would cost much more.
- **Precision vs recall**: High precision means few false blurs; high recall means few missed faces. For privacy, prioritize recall but avoid over-blurring.
- **Automation vs human checks**: Fully automatic scales; human-in-the-loop fixes edge cases.

---

## Questions 
- **How do you get training data?**
  "Start with sampled annotated images, expand with augmentation, and collect user reports to add hard examples for retraining." 

- **How to reduce duplicate boxes?**
  "Use Non-Maximum Suppression: keep the best box and remove overlapping ones." 

- **Which errors are worse?**
  "Missing a face (privacy leak) is usually worse than blurring something harmless — so favor recall with controlled precision." 

- **How to measure success?**
  "Offline: AP/mAP (detection accuracy). Online: user report counts, and manual audits for quality." 

---

## Keywords
- **Object detection:** Finding objects and their locations.
- **Bounding box:** The rectangle that marks an object in an image.
- **Non-Maximum Suppression (NMS):** Remove duplicate overlapping boxes.
- **Data augmentation:** Make more training examples by slightly changing images.
- **Hard negative mining:** Add tricky false-positive cases into training so model learns them.
- **Human-in-the-loop:** Use people to check model outputs and correct labels.


---

## Quick Infos
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
