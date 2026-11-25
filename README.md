# Pet Image Semantic Segmentation Using U-Net

The project aims to develop a computer vision model capable of performing pixel-level segmentation of cats and dogs in images, separating the animal from its surrounding background. The project will investigate the robustness of semantic segmentation under variations in pose, scale, lighting, and image appearance.

## Notebooks

| Notebook | Milestone | What it does |
|---|---|---|
| [`01_dataset_analysis.ipynb`](notebooks/01_dataset_analysis.ipynb) | 1 | Explores the dataset: images, trimaps, class and breed distribution |
| [`02_unet_baseline_and_improvement.ipynb`](notebooks/02_unet_baseline_and_improvement.ipynb) | 2 | Trains the baseline U-Net from scratch, plus augmentation and Dice-loss experiments |
| [`03_improvement_pretrained_encoder.ipynb`](notebooks/03_improvement_pretrained_encoder.ipynb) | 3 | Improves the U-Net with a ResNet-34 encoder pre-trained on ImageNet; ablation, per-image analysis, longer-training analysis and a demo |

Each pixel is labelled **pet**, **background** or **boundary** (the dataset's trimap). Images are resized to 256 × 256.
The train/validation split is 2,944 / 736 images (stratified by breed, from `trainval`); the official `test` split (3,669 images) is used once per run.

## Results (test set)

| Model | mIoU | mDice | IoU pet / background / boundary |
|---|---|---|---|
| U-Net from scratch (Milestone 2 baseline) | 0.744 | 0.842 | 0.820 / 0.900 / 0.511 |
| U-Net + ResNet-34 encoder, random init (ablation) | 0.729 | 0.831 | 0.805 / 0.892 / 0.490 |
| **U-Net + ResNet-34 encoder, ImageNet init (Milestone 3)** | **0.785** | **0.871** | **0.863 / 0.927 / 0.567** |

All three use identical settings (AdamW, lr 1e-3, batch 16, 30 epochs, cross-entropy, no augmentation, seed 42).
Figures are in [`results/`](results/) (`m3_*.png` for Milestone 3).

## How to run

1. Open a notebook in Google Colab (or the VS Code Colab extension) with a **GPU** runtime and a **Python** kernel.
2. Run the cells in order. The dataset is downloaded to the runtime's local disk; checkpoints, result files and figures are saved to `MyDrive/Pet-Image-Segmentation-CV/outputs/`.
3. Finished runs are reloaded from `outputs/` instead of being retrained, so after a disconnect just re-run the cells. Notebook 3 reuses the Milestone 2 baseline saved there by notebook 2.
4. Demo: put your own photos in `MyDrive/Pet-Image-Segmentation-CV/demo/` and run the demo cell in notebook 3.

## Dataset & License

This project uses the **Oxford-IIIT Pet Dataset**, provided by the Visual Geometry Group (VGG) at the University of Oxford.

* **Dataset:** Oxford-IIIT Pet
* **Source:** University of Oxford — Visual Geometry Group
* **License:** [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
* **Dataset page:** https://www.robots.ox.ac.uk/~vgg/data/pets/

The dataset contains images of cats and dogs across 37 pet breed categories, with pixel-level foreground/background trimap annotations. The dataset is available for commercial and research purposes under the CC BY-SA 4.0 license. Copyright of the original images remains with their respective owners.

### Attribution

> Parkhi, O. M., Vedaldi, A., Zisserman, A., & Jawahar, C. V. (2012). *Cats and Dogs*. IEEE Conference on Computer Vision and Pattern Recognition (CVPR).

The dataset is used in this project for academic and research purposes. The dataset itself is **not included in this repository**; users should obtain it from the official source and comply with its license terms.
