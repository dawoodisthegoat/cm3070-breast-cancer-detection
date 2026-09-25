# Deep Learning Breast Cancer Detection on CBIS-DDSM

CM3070 Final Project, BSc Computer Science (University of London).
Template: CM3015 Machine Learning and Neural Networks, Project Idea 2: Deep Learning Breast Cancer Detection.

This project trains CNNs to classify full mammograms from CBIS-DDSM as benign or malignant. The idea is a tool that could support a radiologist as a second reader. Eight models are compared on the same patient-level test set, and the results are checked for calibration, subgroup performance and where the model is looking (Grad-CAM).

## What is in the repo

| Path | Contents |
|---|---|
| `CM3070_Breast_Cancer_Detection.ipynb` | The whole project: data, split, preprocessing, the 8 experiments, evaluation, Grad-CAM and the Gradio demo |
| `results/` | Figures, tables, per-image predictions, training logs, demo images and the weights of the main model |

## How to run it

1. Open the notebook in Google Colab. A GPU runtime is needed for training, but not for evaluation or the demo if the saved results are already on Drive.
2. Add your Kaggle username and key as Colab secrets called `KAGGLE_USERNAME` and `KAGGLE_KEY`. The notebook downloads the [CBIS-DDSM JPEG dataset from Kaggle](https://www.kaggle.com/datasets/debjeetdas/breast-cancer-jpg-image-dataset-of-cbisddsm).
3. Run all. Outputs are saved to `MyDrive/CM3070_project/`. Experiments that have already finished are skipped, so the notebook can carry on after a disconnect.

Training all eight models took about 2.6 hours on a T4 GPU. To run only some of them, set `CONFIG["run_experiments"]`.

## Experiments

| ID | Model | Input | Loss | Augmentation | Why |
|---|---|---|---|---|---|
| B0 | DenseNet121, frozen, 5 epochs | 224 | BCE | No | Draft report baseline, re-run on the new pipeline |
| M1 | Custom CNN from scratch | 512 | Focal | Yes | Does transfer learning help? |
| M2 | DenseNet121 fine-tuned | 512 | BCE | Yes | Does Focal Loss help? |
| M3 | DenseNet121 fine-tuned | 512 | Focal | Yes | Proposed model |
| A1 | DenseNet121 fine-tuned | 512 | Focal | No | Does augmentation help? |
| A2 | DenseNet121 fine-tuned | 224 | Focal | Yes | Does image size matter? |
| A3 | EfficientNetB0 fine-tuned | 512 | Focal | Yes | Different backbone |
| A4 | ResNet50V2 fine-tuned | 512 | Focal | Yes | Different backbone |

## Results

Test set: 458 images from 235 patients that were never used for training or tuning.

| Model | Test AUC (95% CI) |
|---|---|
| B0 draft baseline | 0.679 (0.620–0.733) |
| M1 custom CNN | 0.653 (0.590–0.711) |
| M2 DenseNet121, BCE | 0.748 (0.690–0.805) |
| M3 DenseNet121, Focal Loss (proposed) | 0.730 (0.670–0.789) |
| A1 no augmentation | 0.769 (0.714–0.821) |
| A2 224 px | 0.640 (0.582–0.699) |
| A3 EfficientNetB0 | 0.711 (0.654–0.768) |
| A4 ResNet50V2 | 0.751 (0.694–0.806) |

Main findings:

- Using 512 px images instead of 224 px made the biggest difference, and transfer learning beat training from scratch.
- Focal Loss and augmentation did not improve the results.
- On the same test images, the radiologists' BI-RADS scores reached an AUC of 0.790 against 0.729 for M3. The gap was not statistically significant.
- Splitting by patient matters: a random image split would have put 68% of test images in patients already seen in training.
- Caching the preprocessing made training about 51 times faster than in the draft.
- Grad-CAM points at the lesion more often than chance, but a lot of the heatmap falls outside the breast, which is the main limitation.

## Notes

- The split is saved in `results/tables/split_manifest.csv`, and every number in the report can be recomputed from the saved predictions in `results/preds/`.
- Thresholds and calibration were fitted on the validation set only.
- This is a research prototype trained on old digitised film mammograms that all contain a biopsied lesion. It is not a medical device.
