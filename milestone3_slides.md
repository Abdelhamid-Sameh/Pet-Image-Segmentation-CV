# Milestone 3 — Slide Content (Pet Image Semantic Segmentation Using U-Net)

## Instructions for the slide builder (read first)

- Build a PowerPoint-style deck from the slides below. One `## Slide N` block = one slide.
- **Keep the text exactly as simple as written.** Do not add claims, numbers or results that are not in this file.
- **Style:** match the Milestone 1 and 2 decks: dark navy background, white text, orange highlight for the second line of each title, condensed bold uppercase titles, clean sans-serif body. Plenty of empty space. Maximum about 6 short lines per slide.
- **Figures:** where a slide says `[FIGURE: filename]`, leave a clearly marked image placeholder. I will insert the picture myself. Files: `m3_curves.png`, `m3_confusion_matrices.png`, `m3_per_image_gain.png`, `m3_predictions.png`, `m3_long_curves.png`, `m3_demo.png`.
- **Speaker notes:** put the "Say:" text in the slide's speaker notes, not on the slide.
- The presenter is Abdelhamid Sameh. The audience is a TA grading the final milestone of a university Computer Vision project.

---

## Slide 1 — Title

**Title:** Pet Image *Semantic Segmentation* Using U-Net
**Subtitle:** Milestone 3 — Improvement and Final Results
**Prepared by:** Abdelhamid Sameh

---

## Slide 2 — The problem

**Title:** THE *PROBLEM*

- Input: a photo of a cat or a dog
- Output: a label for **every pixel**: **Pet**, **Background** or **Boundary**
- Goal: separate the pet from the background as accurately as possible

[FIGURE: original image and mask picture from Milestone 1]

**Say:** This is pixel-level segmentation. Every pixel of the photo gets one of three labels.

---

## Slide 3 — Dataset and preprocessing

**Title:** DATA AND *PREPROCESSING*

| Part | Images | Used for |
|---|---|---|
| Train | 2,944 | Learning the weights |
| Validation | 736 | Choosing the best epoch only |
| Test | 3,669 | Final numbers, used once |

- Oxford-IIIT Pet, 37 breeds of cats and dogs
- Resize to 256 × 256, normalise with ImageNet mean and standard deviation
- Same split and preprocessing in every run

**Say:** The test set is never used for any decision. It is only measured once at the end of each run.

---

## Slide 4 — Where Milestone 2 ended

**Title:** WHERE WE *STARTED*

- Baseline: **U-Net trained from scratch**, 7.8 million parameters
- Test mIoU: **0.744**
- Weakest class: **Boundary** (IoU 0.511)
- Typical failure: holes in pets with white fur on a bright background

**Say:** This baseline is the result I had to improve in this milestone.

---

## Slide 5 — The rules for this milestone

**Title:** WHAT COUNTS AS AN *IMPROVEMENT*

Not allowed:
- Changing hyper-parameters (learning rate, batch size, epochs)
- Changing the loss function or the metrics
- Switching to a different model
- Data augmentation

Allowed: **an updated version of the same model**

**Say:** So the improvement has to come from the model itself, with everything else kept exactly the same.

---

## Slide 6 — The improvement

**Title:** THE *IMPROVEMENT*

- Keep the U-Net: same decoder, skip connections, upsampling and output layer
- Upgrade the **encoder** to the residual blocks of **ResNet-34**
- Start the encoder from weights **pre-trained on ImageNet** (transfer learning)

**Why:** with only about 3,000 training images, a model learning from scratch has little to learn from. ImageNet features already recognise edges, textures and fur.

**Say:** The U-Net shape is unchanged. Only the encoder blocks are upgraded and start from knowledge learned on 1.2 million ImageNet images.

---

## Slide 7 — The updated architecture

**Title:** THE UPDATED *U-NET*

| Level | Encoder (ResNet-34) | Size |
|---|---|---|
| 1 | Full-resolution conv block | 256 × 256 |
| 2 | ResNet stem | 128 × 128 |
| 3 | Residual stage 1 | 64 × 64 |
| 4 | Residual stage 2 | 32 × 32 |
| 5 | Residual stage 3 | 16 × 16 |
| Bottom | Residual stage 4 | 8 × 8 |

- Decoder: the same U-Net blocks, with a skip connection at every level
- 24.5 million parameters

**Say:** Each encoder level passes its features straight to the decoder through a skip connection, exactly like the original U-Net.

---

## Slide 8 — Experimental setup

**Title:** EXPERIMENTAL *SETUP*

| Setting | Value (unchanged from Milestone 2) |
|---|---|
| Optimizer | AdamW, learning rate 1e-3, weight decay 1e-4 |
| Schedule | Cosine decay |
| Batch size / epochs | 16 / 30 |
| Loss | Cross-entropy |
| Augmentation | None |
| Seed | 42 |
| Model selection | Best validation mIoU |
| Hardware | Google Colab GPU, about 17 min per run |

**Say:** Every setting is identical to Milestone 2, so any difference comes from the model.

---

## Slide 9 — The experiments

**Title:** THE *EXPERIMENTS*

| Run | Encoder | Pre-trained? | Purpose |
|---|---|---|---|
| **A** | Original U-Net | No | Milestone 2 baseline |
| **D** | ResNet-34 | No | Is it just the bigger encoder? |
| **E** | ResNet-34 | **Yes (ImageNet)** | **The improvement** |

**Say:** Run D has exactly the same architecture as E but starts from random weights. Comparing D and E tells us whether the gain comes from the bigger encoder or from pre-training.

---

## Slide 10 — Results

**Title:** *RESULTS* — TEST SET

| Run | mIoU | mDice | IoU Pet | IoU Background | IoU Boundary |
|---|---|---|---|---|---|
| A — Baseline | 0.744 | 0.842 | 0.820 | 0.900 | 0.511 |
| D — ResNet-34, scratch | 0.729 | 0.831 | 0.805 | 0.892 | 0.490 |
| **E — ResNet-34, ImageNet** | **0.785** | **0.871** | **0.863** | **0.927** | **0.567** |

- **+4.2 mIoU points** over the baseline
- Every class improves; Boundary improves the most (+5.5 points)

*(Validation mIoU: A 0.731, D 0.719, E 0.777)*

**Say:** The improvement is consistent on the validation set and the test set, and on every class.

---

## Slide 11 — Why it works

**Title:** WHY IT *WORKS*

- D (bigger encoder, no pre-training) is slightly **worse** than the baseline
- E (same encoder, pre-trained) is clearly **better**
- So the gain comes from **pre-training**, not from model size

**Say:** A bigger model on its own does not help with only 3,000 training images. What helps is starting from features that were already learned on ImageNet.

---

## Slide 12 — Training curves

**Title:** TRAINING *CURVES*

[FIGURE: m3_curves.png]

- E reaches 0.66 validation mIoU after **one** epoch (baseline: 0.48)
- E is best at epoch 19
- After about epoch 13 its validation loss rises slightly: mild overfitting

**Say:** Pre-training gives the model a huge head start. It reaches 0.72 validation mIoU at epoch 4; the baseline needs about 20 epochs for that.

---

## Slide 13 — Where the errors are

**Title:** WHERE THE *ERRORS* ARE

[FIGURE: m3_confusion_matrices.png]

- Boundary pixels found: **64% → 69%**
- Pet pixels mistaken for background: **4% → 2%**
- Boundary is still the hardest class

**Say:** The improved model makes fewer mistakes everywhere, but the thin, uncertain boundary band remains the hardest part.

---

## Slide 14 — Is the improvement real?

**Title:** IS THE IMPROVEMENT *REAL*?

[FIGURE: m3_per_image_gain.png]

- Average gain per test image: **+0.042 mIoU**
- 95% confidence interval: **+0.040 to +0.044**
- Better on **77%** of test images
- Cats +4.9 points, dogs +3.8 points

**Say:** I compared both models on every one of the 3,669 test images. The gain is not caused by a few lucky images. The interval covers test-image sampling, not training randomness.

---

## Slide 15 — Example predictions

**Title:** EXAMPLE *PREDICTIONS*

[FIGURE: m3_predictions.png]

- The white dog that had holes in Milestone 2 is now filled in
- Cleaner outlines and fewer stray blobs
- Same 6 test images as in Milestone 2

**Say:** The columns are image, ground truth, then the baseline, the scratch ResNet and the improved model.

---

## Slide 16 — Extra analysis: longer training

**Title:** EXTRA ANALYSIS: *LONGER TRAINING*

[FIGURE: m3_long_curves.png]

| Run | Test mIoU, 30 epochs | Test mIoU, 60 epochs + early stopping |
|---|---|---|
| A — Baseline | 0.744 | 0.746 |
| C — Augmentation + Dice (Milestone 2) | 0.737 | 0.771 |
| D — ResNet-34, scratch | 0.729 | 0.732 |
| E — ResNet-34, ImageNet | 0.785 | 0.790 |

- Not an improvement (more epochs is not allowed), only a check of two questions
- Milestone 2 guess confirmed: augmentation needed more training (C: +3.4 points)
- D was not just under-trained, so pre-training really is the reason for E's gain
- E after 30 epochs still beats C after 60 epochs

**Say:** In Milestone 2 I guessed that augmentation hurt because 30 epochs was too short. This confirms it. It also confirms that the comparison between D and E is fair.

---

## Slide 17 — Demo

**Title:** *DEMO*

[FIGURE: m3_demo.png]

- The trained model segments a photo it has never seen
- Shown: photo, predicted mask, pet highlighted

**Say:** I will now run the model live on a new photo.

---

## Slide 18 — Pipeline and code

**Title:** PIPELINE AND *CODE*

- **Notebook 1:** dataset exploration
- **Notebook 2:** baseline U-Net and the Milestone 2 experiments
- **Notebook 3:** the improvement, the analysis and the demo
- Pipeline: load → resize and normalise → model → most likely class per pixel → mask
- Every run saves its model and results, so finished runs are reloaded, not retrained

**Say:** All code is on GitHub. Every number in this presentation comes from these notebooks.

---

## Slide 19 — Limitations

**Title:** *LIMITATIONS*

- One random seed per run (re-runs moved results by only about 0.3–0.4 points)
- Measured on 256 × 256 resized masks, not the original image size
- Boundary is still the weakest class (IoU 0.567)
- Mild overfitting after about epoch 13
- Validation picks the best epoch, so test numbers are the honest ones

**Say:** The gain of 4.2 points is about ten times larger than the run-to-run variation I observed, but a proper study with several seeds would make that certain.

---

## Slide 20 — Future work

**Title:** *FUTURE WORK*

1. Combine the pre-trained encoder with augmentation and longer training — both helped on their own
2. Repeat runs with several random seeds
3. Evaluate at the original image resolution
4. Focus on the boundary: higher input resolution or boundary-aware training

**Say:** These go beyond the rules of this milestone, but they are the natural next steps.

---

## Slide 21 — Thank you

**Title:** THANK YOU !
**Subtitle:** QUESTIONS?

---

# Q&A cheat sheet (NOT a slide — for the presenter only)

- **Why is D worse than the baseline?** It is three times bigger and starts from random weights. With only about 3,000 training images, extra size alone does not help.
- **How do you know the gain is not luck?** It holds on validation and test, on every class, on 77% of test images, and it is about ten times larger than the variation between re-runs.
- **Why did you not use augmentation?** The milestone rules do not allow it. The extra analysis shows it helps once training is longer, so it is in future work.
- **Why is the best epoch 19 and not 30?** The pre-trained model learns fast. After about epoch 13 it starts to overfit, and model selection keeps the best validation epoch.
- **How do you avoid test leakage?** The test set is only measured once per run, with the model chosen on the validation set.
- **Why is the test score higher than validation?** All runs show it. It is a different set of images, so a small gap is normal.
- **Why does pre-training help the boundary most?** Pre-trained features already detect edges well, and the boundary is exactly the edge of the pet.
- **Why train for 60 epochs if it is not allowed?** Only as analysis, to test the Milestone 2 hypothesis and to check that D was trained fairly. It is not the improvement.
