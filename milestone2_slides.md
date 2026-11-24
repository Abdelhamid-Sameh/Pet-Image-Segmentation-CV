# Milestone 2 — Slide Content (Pet Image Semantic Segmentation Using U-Net)

## Instructions for the slide builder (read first)

- Build a PowerPoint-style deck from the slides below. One `## Slide N` block = one slide.
- **Keep the text exactly as simple as written.** Do not add claims, numbers or results that are not in this file.
- **Style:** match the Milestone 1 deck: dark navy background, white text, orange highlight for the second line of each title (for example "RESULTS" white + "AND DISCUSSION" orange), condensed bold uppercase titles, clean sans-serif body. Plenty of empty space. Maximum about 6 short lines per slide.
- **Figures:** where a slide says `[FIGURE: filename]`, leave a clearly marked image placeholder. I will insert the picture myself. The files are `curves.png`, `confusion_matrices.png`, `predictions.png`.
- **Speaker notes:** put the "Say:" text in the slide's speaker notes, not on the slide.
- The presenter is Abdelhamid Sameh. The audience is a TA grading a university Computer Vision project.

---

## Slide 1 — Title

**Title:** Pet Image *Semantic Segmentation* Using U-Net
**Subtitle:** Milestone 2 — Experiments and Results
**Prepared by:** Abdelhamid Sameh

---

## Slide 2 — Recap: the problem

**Title:** THE *PROBLEM*

- Input: a photo of a cat or a dog
- Output: a label for **every pixel**
- Three classes: **Pet**, **Background**, **Ambiguous boundary**
- Model: **U-Net**, trained from scratch

[FIGURE: use the original-image-and-mask picture from Milestone 1 if available]

**Say:** In Milestone 1 I defined the task. In this milestone I trained the model and measured how well it works.

---

## Slide 3 — Dataset

**Title:** THE *DATASET*

- **Oxford-IIIT Pet**, from the University of Oxford (VGG)
- 37 breeds: 12 cat and 25 dog
- Every image has a pixel-level mask (trimap)
- License: CC BY-SA 4.0

**Say:** I use this dataset because it is a standard benchmark focused on cats and dogs, with an exact mask for every image.

---

## Slide 4 — Other datasets considered

**Title:** OTHER *DATASETS* I CONSIDERED

| Dataset | Why I did not choose it |
|---|---|
| PASCAL VOC 2012 | Cats and dogs are only 2 of 20 classes, so fewer pet images |
| MS COCO | Very large, mostly non-pet images, coarser masks |
| Stanford Dogs | Bounding boxes only, no pixel masks |

**Footer line:** Idea for later: train on Oxford Pets, test on VOC cats/dogs to check robustness.

**Say:** Oxford-IIIT Pet is the best fit. A second dataset would be useful later to test how well the model generalises.

---

## Slide 5 — Data split (what I am doing)

**Title:** HOW I *USE THE DATA*

| Part | Images | Used for |
|---|---|---|
| Train | 2,944 | Learning the weights |
| Validation | 736 | Choosing the best epoch only |
| Test | 3,669 | Final numbers, used once |

- Train + validation come from the 3,680-image *trainval* split (80/20, same share of each breed)
- Test is the official *test* split
- I am **training and evaluating**. I am not tuning hyper-parameters.

**Say:** The validation set only picks which epoch to keep. The test set is touched once, at the end, so the final numbers are honest.

---

## Slide 6 — Data preprocessing

**Title:** DATA *PREPROCESSING*

- Resize every image to **256 × 256** (images: bilinear, masks: nearest, so labels stay exact)
- Mask labels 1 / 2 / 3 become **0 = Pet, 1 = Background, 2 = Boundary**
- Normalise with ImageNet mean and standard deviation
- Images are decoded once and kept in memory

**Say:** These steps are the same in every experiment.

---

## Slide 7 — The model

**Title:** THE *MODEL*

- **U-Net**, trained **from scratch** (random weights, no pre-training)
- Encoder: 4 down-steps (convolutions + max-pooling)
- Decoder: 4 up-steps (transposed convolutions)
- Skip connections keep fine detail
- Channels: 32 → 64 → 128 → 256 → 512
- 7.76 million parameters

**Say:** The skip connections are what let U-Net draw sharp pixel-level outlines.

---

## Slide 8 — Training settings

**Title:** TRAINING *SETTINGS*

| Setting | Value |
|---|---|
| Optimizer | AdamW (learning rate 1e-3, weight decay 1e-4) |
| Learning-rate schedule | Cosine decay |
| Batch size | 16 |
| Epochs | 30 |
| Precision | Mixed precision (faster) |
| Random seed | 42 (same in every run) |
| Hardware | Google Colab GPU, about 25 min per run |
| Model selection | Best validation mIoU |

**Say:** Every run uses exactly the same settings. Only the one thing I change on the next slide differs.

---

## Slide 9 — The experiments

**Title:** THE *EXPERIMENTS*

| Run | Augmentation | Loss | Purpose |
|---|---|---|---|
| **A — Baseline** | No | Cross-Entropy | The required run |
| **B** | Yes | Cross-Entropy | Effect of augmentation alone |
| **C** | Yes | Cross-Entropy + Dice | My first improvement attempt |

- Augmentation: random crop, horizontal flip, rotation up to 15°, colour jitter
- Dice loss: focuses on overlap, helps with unbalanced classes

**Say:** A is the main run. B and C test the two improvements from my Milestone 1 plan, changing one thing at a time.

---

## Slide 10 — Metrics

**Title:** HOW I *MEASURE*

- **IoU** = overlap ÷ union (TP / (TP + FP + FN))
- **Dice** = 2 × overlap ÷ (sum of sizes) (2TP / (2TP + FP + FN))
- **mIoU** = average IoU of the 3 classes (main number)
- Also: IoU for each class, and a confusion matrix

**Say:** mIoU is the standard metric for segmentation. I also look per class because the classes behave very differently.

---

## Slide 11 — Results

**Title:** *RESULTS* — TEST SET

| Run | mIoU | mDice | IoU Pet | IoU Background | IoU Boundary |
|---|---|---|---|---|---|
| **A — Baseline** | **0.744** | **0.842** | **0.820** | **0.900** | 0.511 |
| B — Aug | 0.726 | 0.829 | 0.799 | 0.888 | 0.491 |
| C — Aug + Dice | 0.737 | 0.838 | 0.803 | 0.893 | **0.517** |

- The baseline reaches **0.744 mIoU** on the test set
- Boundary is by far the hardest class

*(Validation mIoU: A 0.731, B 0.710, C 0.719)*

**Say:** The baseline is already decent. My first improvement attempt did not beat it. That is an honest result, and I will explain why on the next slides.

---

## Slide 12 — Training curves

**Title:** *TRAINING* CURVES

[FIGURE: curves.png]

- Baseline levels off around epoch 26
- B and C were **still improving** at epoch 30

**Say:** The augmented runs learn more slowly. They had not finished learning when training stopped.

---

## Slide 13 — Where the errors are

**Title:** WHERE THE *ERRORS* ARE

[FIGURE: confusion_matrices.png]

- Pet and Background are mostly correct (about 90% or more)
- Only **64–68%** of true Boundary pixels are found
- The rest are mixed up with Pet or Background

**Say:** The boundary is a thin, uncertain band. A shift of a few pixels already costs a lot of IoU, so it is the hardest class.

---

## Slide 14 — Example predictions

**Title:** EXAMPLE *PREDICTIONS*

[FIGURE: predictions.png]

- Good outlines on clear photos
- Pets with white fur on a bright background get **holes**
- Augmented models (B, C) show some noisy edges near the image border

**Say:** The columns are image, ground truth, then A, B and C. The noisy borders in B and C are something I want to investigate. I think the rotation padding may be the cause, but I have not tested that.

---

## Slide 15 — Discussion

**Title:** *DISCUSSION*

- Augmentation alone **lowered** mIoU by about 1.8 points
- Adding Dice recovered about 1.1 points, but did not beat the baseline
- Difference between A and C is small (about 0.6 points) — with one run each I cannot say it is real
- Likely reason: 30 epochs is too short for augmented training *(not yet tested)*

**Limitations:**
- One random seed per run
- Boundary labels are ambiguous by nature
- Trained from scratch on only about 3,000 images

**Say:** I am being careful: these are my best explanations, not proven facts.

---

## Slide 16 — Proposed improvements

**Title:** PROPOSED *IMPROVEMENTS*

1. **Pre-trained encoder** (U-Net with an ImageNet ResNet) — a from-scratch model has little data to learn from
2. **Train the augmented models longer** — their curves were still rising
3. **Boundary-focused loss or weighting** — target the weakest class
4. **Several random seeds** — to tell real gains from noise

**Say:** The first one is my main proposal and the one I expect to help most. I will implement and test it in the next milestone.

---

## Slide 17 — Next steps

**Title:** *NEXT STEPS*

- Implement the pre-trained encoder and compare with the baseline
- Re-run augmented training for more epochs
- Repeat key runs with different seeds
- Optional: test on another dataset (VOC cats/dogs)

---

## Slide 18 — Thank you

**Title:** THANK YOU !
**Subtitle:** QUESTIONS?

---

# Q&A cheat sheet (NOT a slide — for the presenter only)

- **Are you training, evaluating or tuning?** Training and evaluating. Validation only picks the best epoch. Hyper-parameters were fixed in advance. Test is used once.
- **Did you find other datasets?** Yes: PASCAL VOC, MS COCO and Stanford Dogs. Oxford-IIIT Pet fits best because every image has an exact pet mask.
- **Why did augmentation not help?** Most likely the model needs more than 30 epochs when the training data is made harder. The curves were still rising. I have not tested this yet.
- **Why is the boundary IoU low?** The boundary is a thin band with uncertain ground truth, so small pixel shifts cost a lot of IoU. About a third of boundary pixels get confused with Pet or Background.
- **Is C really worse than A?** The gap is about 0.6 points on test. With one seed per run I cannot claim a real difference.
- **Why train from scratch?** It gives a clean baseline. A pre-trained encoder is my main proposed improvement.
- **Why is test mIoU higher than validation?** All three runs show about 1 point higher on test. It is a different set of images, so a small gap is normal.
