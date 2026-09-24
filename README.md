# Deep Learning Breast Cancer Detection on CBIS-DDSM

CM3070 Final Project (University of London BSc Computer Science).
Template: **CM3015 Machine Learning and Neural Networks — Project Idea 2 (3.2): Deep Learning Breast Cancer Detection.**

A benign-vs-malignant classifier for full-field mammograms from CBIS-DDSM, designed as a decision-support "second reader". The project compares a succession of CNNs on one locked, patient-level test set and evaluates them for discrimination, operating-point performance, calibration, subgroup behaviour and Grad-CAM localisation.

## Contents

| Path | What it is |
|---|---|
| `CM3070_Breast_Cancer_Detection.ipynb` | The full pipeline: data, locked split, preprocessing cache, 8 experiments, evaluation, Grad-CAM and the Gradio demo |
| `results/` | Figures, tables, per-image predictions, training logs, demo examples and the proposed model's weights |

## Running it

1. Open the notebook in Google Colab and select a GPU runtime (T4 or better).
2. Add Kaggle credentials as Colab secrets `KAGGLE_USERNAME` and `KAGGLE_KEY`. The notebook downloads the CBIS-DDSM JPEG export ([Kaggle, CC0](https://www.kaggle.com/datasets/debjeetdas/breast-cancer-jpg-image-dataset-of-cbisddsm)).
3. Run all. Outputs are written to `MyDrive/CM3070_project/`. Finished experiments are skipped on re-run, so the notebook can be resumed after a disconnect.

The full experiment set took about 2.6 GPU-hours on a T4. `CONFIG["run_experiments"]` selects a subset.

## Experiments

| ID | Model | Input | Loss | Augmentation | Purpose |
|---|---|---|---|---|---|
| B0 | DenseNet121, frozen, 5 epochs | 224 | BCE | – | Draft-report baseline re-run on the new pipeline |
| M1 | Custom CNN (from scratch) | 512 | Focal | ✓ | Transfer learning vs. no transfer |
| M2 | DenseNet121, fine-tuned | 512 | BCE | ✓ | Effect of fine-tuning |
| **M3** | **DenseNet121, fine-tuned** | **512** | **Focal** | ✓ | **Proposed model** |
| A1 | DenseNet121, fine-tuned | 512 | Focal | – | Ablation: augmentation |
| A2 | DenseNet121, fine-tuned | 224 | Focal | ✓ | Ablation: resolution |
| A3 | EfficientNetB0, fine-tuned | 512 | Focal | ✓ | Alternative backbone |
| A4 | ResNet50V2, fine-tuned | 512 | Focal | ✓ | Alternative backbone |

## Results (locked test set: 458 images, 235 patients)

| Model | Test AUC (95% CI) | Sensitivity / specificity at Youden threshold |
|---|---|---|
| B0: draft baseline (frozen DenseNet121, 224 px) | 0.679 (0.620–0.733) | 47.2% / 76.2% |
| M1: custom CNN from scratch | 0.653 (0.590–0.711) | 64.8% / 54.7% |
| M2: DenseNet121 fine-tuned, BCE | 0.748 (0.690–0.805) | 74.1% / 64.9% |
| **M3: DenseNet121 fine-tuned, Focal Loss (proposed)** | **0.730 (0.670–0.789)** | **68.4% / 67.2%** |
| A1: M3 without augmentation | 0.769 (0.714–0.821) | 72.0% / 64.9% |
| A2: M3 at 224 px | 0.640 (0.582–0.699) | 57.5% / 61.5% |
| A3: EfficientNetB0 fine-tuned | 0.711 (0.654–0.768) | 69.9% / 54.3% |
| A4: ResNet50V2 fine-tuned | 0.751 (0.694–0.806) | 68.9% / 65.3% |

Other findings:

* **Radiologist reference:** the original radiologists' BI-RADS scores reach AUC 0.790 on the same test images, against 0.729 for M3. The difference is not statistically significant (p = 0.079).
* **What mattered:** 512 px input beat 224 px (+0.089 AUC, p < 0.001), and ImageNet transfer beat training from scratch (+0.077, p = 0.046). Focal Loss and augmentation did not improve AUC.
* **Leakage audit:** a naive image-level split would have put 68% of test images in the same patients as training; the patient-level split puts 0%.
* **Speed:** caching the preprocessing cut the draft configuration from 1,033 s to 20 s per epoch.
* **Explainability:** Grad-CAM peaks fall on the radiologist-annotated lesion in 11.4% of test images, against 6.4% chance. Around 60% of heatmap energy falls outside the breast, which suggests some reliance on non-pathological cues. This is the main limitation.

All figures and tables are in `results/`.

## Reproducibility

* Patient-level `GroupShuffleSplit` (seed 42), verified to have zero patient overlap. The exact split is saved in `results/tables/split_manifest.csv`.
* Global seeds are fixed. GPU kernels are not fully deterministic, so retraining can move metrics slightly; the saved per-image predictions reproduce every reported number exactly.
* All thresholds and calibration parameters are fitted on the validation set only.

## Disclaimer

Research prototype trained on digitised film mammograms that all contain a biopsied lesion. It is not a medical device and must not be used for clinical decisions.
