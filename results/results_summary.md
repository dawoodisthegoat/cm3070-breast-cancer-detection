# Results summary

Proposed model: `M3_densenet_ft_focal_512`

Thresholds (validation): `{"0.50": 0.5, "youden": 0.4982281625270843, "sens90": 0.4061805009841919}`

Preprocessing build times (s): `{"images_512": 574, "images_224": 4, "masks_test_512": 1, "masks_test_512_v2": 41}`

BI-RADS: ΔAUC model − radiologists = -0.061 (95% CI -0.130 to +0.008, p = 0.079); model sensitivity at BI-RADS≥4 specificity = 86.3%

## Split summary

| split   |   images |   patients |   malignant |   malignant_% |
|:--------|---------:|-----------:|------------:|--------------:|
| train   |     2190 |       1096 |         977 |          44.6 |
| val     |      455 |        235 |         205 |          45.1 |
| test    |      458 |        235 |         193 |          42.1 |

## Leakage audit

| Split strategy                     | Test images sharing a patient with training   |   Near-duplicate pairs crossing partitions |
|:-----------------------------------|:----------------------------------------------|-------------------------------------------:|
| Patient-level (used)               | 0                                             |                                          0 |
| Naive image-level (counterfactual) | 317 (68.0%)                                   |                                         29 |

## Model sizes

| Backbone       |   Total parameters | Feature map at 512 px   | Trainable (head phase)   |   Trainable (fine-tune / full) |
|:---------------|-------------------:|:------------------------|:-------------------------|-------------------------------:|
| custom         |            1251489 | 16×16×256               | -                        |                        1248993 |
| densenet121    |            7304257 | 16×16×1024              | 264705                   |                        2394625 |
| efficientnetb0 |            4382884 | 16×16×1280              | 330753                   |                        3460637 |
| resnet50v2     |           24097793 | 16×16×2048              | 528897                   |                       15479297 |

## Experiments

| ID                             | Backbone       |   Input | Loss   | Augment   |   Batch | Phases                                               | Role                                            |
|:-------------------------------|:---------------|--------:|:-------|:----------|--------:|:-----------------------------------------------------|:------------------------------------------------|
| B0_densenet_frozen_bce_224     | densenet121    |     224 | bce    | False     |      32 | head (lr 0.0001, ≤5 ep)                              | Draft baseline re-run (identical configuration) |
| M1_custom_cnn_focal_512        | custom         |     512 | focal  | True      |      16 | full (lr 0.001, ≤40 ep)                              | Custom CNN trained from scratch                 |
| M2_densenet_ft_bce_512         | densenet121    |     512 | bce    | True      |      16 | head (lr 0.001, ≤8 ep) → finetune (lr 3e-05, ≤25 ep) | Fine-tuned DenseNet121, binary cross-entropy    |
| M3_densenet_ft_focal_512       | densenet121    |     512 | focal  | True      |      16 | head (lr 0.001, ≤8 ep) → finetune (lr 3e-05, ≤25 ep) | Proposed: fine-tuned DenseNet121, focal loss    |
| A1_densenet_ft_focal_noaug_512 | densenet121    |     512 | focal  | False     |      16 | head (lr 0.001, ≤8 ep) → finetune (lr 3e-05, ≤25 ep) | Ablation: no augmentation                       |
| A2_densenet_ft_focal_224       | densenet121    |     224 | focal  | True      |      16 | head (lr 0.001, ≤8 ep) → finetune (lr 3e-05, ≤25 ep) | Ablation: 224 px input                          |
| A3_efficientnetb0_ft_focal_512 | efficientnetb0 |     512 | focal  | True      |      16 | head (lr 0.001, ≤8 ep) → finetune (lr 3e-05, ≤25 ep) | Alternative backbone: EfficientNetB0            |
| A4_resnet50v2_ft_focal_512     | resnet50v2     |     512 | focal  | True      |      16 | head (lr 0.001, ≤8 ep) → finetune (lr 3e-05, ≤25 ep) | Alternative backbone: ResNet50V2                |

## Training efficiency

| Run                                            |   Input |   Epochs |   Mean epoch (s) |   Total (min) |
|:-----------------------------------------------|--------:|---------:|-----------------:|--------------:|
| Draft pipeline (per-epoch JPEG decode + CLAHE) |     224 |        5 |           1032.8 |          86.1 |
| B0_densenet_frozen_bce_224                     |     224 |        5 |             20.1 |           1.7 |
| M1_custom_cnn_focal_512                        |     512 |       29 |             35.3 |          17.3 |
| M2_densenet_ft_bce_512                         |     512 |       31 |             59.4 |          31.7 |
| M3_densenet_ft_focal_512                       |     512 |       33 |             58.3 |          33.1 |
| A1_densenet_ft_focal_noaug_512                 |     512 |       22 |             58   |          22.2 |
| A2_densenet_ft_focal_224                       |     224 |       17 |             16.5 |           5.3 |
| A3_efficientnetb0_ft_focal_512                 |     512 |       30 |             37   |          19.4 |
| A4_resnet50v2_ft_focal_512                     |     512 |       26 |             53.2 |          23.8 |

## Test metrics (locked test set)

| Model                          |   Val AUC | Test AUC (95% CI)   | Test PR-AUC (95% CI)   | Sens % @ Youden   | Spec % @ Youden   | PPV % @ Youden   | Sens % @ Sens≥90%   | Spec % @ Sens≥90%   | PPV % @ Sens≥90%   |
|:-------------------------------|----------:|:--------------------|:-----------------------|:------------------|:------------------|:-----------------|:--------------------|:--------------------|:-------------------|
| B0_densenet_frozen_bce_224     |     0.68  | 0.679 (0.620–0.733) | 0.571 (0.477–0.663)    | 47.2 (39.6–54.9)  | 76.2 (69.9–81.8)  | 59.1 (49.0–68.2) | 87.6 (81.6–92.7)    | 33.6 (27.2–39.8)    | 49.0 (41.9–55.9)   |
| M1_custom_cnn_focal_512        |     0.719 | 0.653 (0.590–0.711) | 0.568 (0.485–0.659)    | 64.8 (55.8–73.1)  | 54.7 (46.5–62.6)  | 51.0 (43.0–59.2) | 84.5 (77.7–90.6)    | 34.0 (26.4–41.9)    | 48.2 (40.8–55.5)   |
| M2_densenet_ft_bce_512         |     0.752 | 0.748 (0.690–0.805) | 0.674 (0.589–0.760)    | 74.1 (66.7–81.7)  | 64.9 (58.0–71.6)  | 60.6 (52.6–68.5) | 90.2 (84.7–94.9)    | 38.9 (31.1–46.7)    | 51.8 (44.9–59.1)   |
| M3_densenet_ft_focal_512       |     0.752 | 0.730 (0.670–0.789) | 0.658 (0.570–0.746)    | 68.4 (60.8–76.4)  | 67.2 (60.6–73.6)  | 60.3 (52.2–68.4) | 89.1 (83.7–94.0)    | 35.1 (27.0–43.3)    | 50.0 (43.3–57.1)   |
| A1_densenet_ft_focal_noaug_512 |     0.785 | 0.769 (0.714–0.821) | 0.701 (0.621–0.778)    | 72.0 (64.1–79.5)  | 64.9 (57.1–72.2)  | 59.9 (51.6–68.0) | 86.0 (79.7–91.9)    | 51.3 (42.9–59.5)    | 56.3 (48.8–64.0)   |
| A2_densenet_ft_focal_224       |     0.676 | 0.640 (0.582–0.699) | 0.546 (0.457–0.634)    | 57.5 (49.5–65.3)  | 61.5 (54.9–68.0)  | 52.1 (43.9–60.2) | 83.9 (78.0–89.4)    | 30.2 (23.0–37.6)    | 46.7 (39.9–53.5)   |
| A3_efficientnetb0_ft_focal_512 |     0.78  | 0.711 (0.654–0.768) | 0.640 (0.548–0.735)    | 69.9 (62.4–77.3)  | 54.3 (46.6–62.0)  | 52.7 (44.6–60.9) | 89.1 (83.9–93.8)    | 39.2 (31.2–47.4)    | 51.7 (44.4–59.0)   |
| A4_resnet50v2_ft_focal_512     |     0.739 | 0.751 (0.694–0.806) | 0.706 (0.619–0.788)    | 68.9 (60.7–76.9)  | 65.3 (58.5–71.9)  | 59.1 (50.5–67.4) | 88.6 (83.4–93.5)    | 38.9 (31.5–46.1)    | 51.4 (44.2–58.3)   |

## Paired AUC comparison vs proposed

| Proposed vs                    |   ΔAUC | 95% CI           |   p (paired bootstrap) |
|:-------------------------------|-------:|:-----------------|-----------------------:|
| B0_densenet_frozen_bce_224     |  0.051 | -0.006 to +0.108 |                  0.074 |
| M1_custom_cnn_focal_512        |  0.077 | +0.002 to +0.152 |                  0.046 |
| M2_densenet_ft_bce_512         | -0.018 | -0.036 to +0.000 |                  0.053 |
| A1_densenet_ft_focal_noaug_512 | -0.039 | -0.077 to -0.000 |                  0.048 |
| A2_densenet_ft_focal_224       |  0.089 | +0.036 to +0.142 |                  0     |
| A3_efficientnetb0_ft_focal_512 |  0.019 | -0.027 to +0.068 |                  0.459 |
| A4_resnet50v2_ft_focal_512     | -0.022 | -0.061 to +0.022 |                  0.303 |

## Calibration

| Model                          |   Temperature |   ECE before |   ECE after |   Brier before |   Brier after |   NLL before |   NLL after |
|:-------------------------------|--------------:|-------------:|------------:|---------------:|--------------:|-------------:|------------:|
| B0_densenet_frozen_bce_224     |         1.082 |        0.04  |       0.033 |          0.223 |         0.223 |        0.636 |       0.635 |
| M1_custom_cnn_focal_512        |         0.334 |        0.078 |       0.098 |          0.235 |         0.238 |        0.662 |       0.67  |
| M2_densenet_ft_bce_512         |         1.79  |        0.113 |       0.074 |          0.214 |         0.204 |        0.627 |       0.591 |
| M3_densenet_ft_focal_512       |         0.596 |        0.071 |       0.072 |          0.212 |         0.21  |        0.613 |       0.603 |
| A1_densenet_ft_focal_noaug_512 |         0.763 |        0.044 |       0.051 |          0.192 |         0.19  |        0.559 |       0.552 |
| A2_densenet_ft_focal_224       |         0.69  |        0.067 |       0.059 |          0.233 |         0.234 |        0.658 |       0.66  |
| A3_efficientnetb0_ft_focal_512 |         0.496 |        0.099 |       0.115 |          0.218 |         0.227 |        0.625 |       0.654 |
| A4_resnet50v2_ft_focal_512     |         0.969 |        0.071 |       0.067 |          0.2   |         0.2   |        0.591 |       0.591 |

## BI-RADS radiologist reference

| Reader                          | AUC (95% CI)        | Operating point     | Sensitivity %    | Specificity %    |
|:--------------------------------|:--------------------|:--------------------|:-----------------|:-----------------|
| Radiologists (BI-RADS, ordinal) | 0.790 (0.738–0.837) | BI-RADS ≥ 4         | 90.1 (84.2–95.5) | 42.8 (33.9–51.2) |
| M3_densenet_ft_focal_512        | 0.729 (0.668–0.784) | Youden (validation) | 66.5 (57.9–74.4) | 68.7 (62.1–75.5) |

## Subgroups (proposed model)

| Group          | Subgroup      |   Images |   Malignant | AUC (95% CI)        | Sensitivity %     | Specificity %    |
|:---------------|:--------------|---------:|------------:|:--------------------|:------------------|:-----------------|
| Breast density | Density 1     |       54 |          17 | 0.800 (0.603–0.949) | 76.5 (50.0–100.0) | 75.7 (58.5–91.4) |
| Breast density | Density 2     |      197 |          98 | 0.720 (0.632–0.812) | 73.5 (62.4–84.6)  | 58.6 (47.5–68.8) |
| Breast density | Density 3     |      138 |          56 | 0.738 (0.623–0.839) | 60.7 (42.9–77.8)  | 72.0 (59.5–83.3) |
| Breast density | Density 4     |       69 |          22 | 0.632 (0.482–0.773) | 59.1 (33.3–82.6)  | 70.2 (56.9–83.0) |
| Lesion type    | Calcification |      217 |          91 | 0.748 (0.665–0.822) | 71.4 (59.8–81.9)  | 68.3 (58.1–77.4) |
| Lesion type    | Mass          |      241 |         102 | 0.712 (0.635–0.789) | 65.7 (54.2–76.6)  | 66.2 (58.0–74.8) |
| View           | CC            |      204 |          89 | 0.720 (0.647–0.789) | 68.5 (58.5–77.6)  | 65.2 (56.2–74.4) |
| View           | MLO           |      254 |         104 | 0.738 (0.670–0.797) | 68.3 (58.1–75.8)  | 68.7 (61.0–75.6) |
| Subtlety       | 1–2 (subtle)  |       86 |          42 | 0.623 (0.470–0.763) | 64.3 (45.0–81.6)  | 50.0 (32.5–66.7) |
| Subtlety       | 3             |      138 |          52 | 0.761 (0.647–0.853) | 69.2 (54.3–82.0)  | 74.4 (64.8–83.5) |
| Subtlety       | 4–5 (obvious) |      232 |          99 | 0.745 (0.662–0.821) | 69.7 (57.6–80.7)  | 68.4 (59.3–77.1) |

## Multi-view fusion

| Model                          | Image-level AUC     | Breast-level AUC    |   Change |   Breasts |   Breasts with ≥2 images |
|:-------------------------------|:--------------------|:--------------------|---------:|----------:|-------------------------:|
| B0_densenet_frozen_bce_224     | 0.679 (0.620–0.733) | 0.696 (0.634–0.757) |    0.017 |       257 |                      189 |
| M1_custom_cnn_focal_512        | 0.653 (0.590–0.711) | 0.647 (0.578–0.709) |   -0.006 |       257 |                      189 |
| M2_densenet_ft_bce_512         | 0.748 (0.690–0.805) | 0.748 (0.688–0.805) |    0     |       257 |                      189 |
| M3_densenet_ft_focal_512       | 0.730 (0.670–0.789) | 0.731 (0.670–0.791) |    0.001 |       257 |                      189 |
| A1_densenet_ft_focal_noaug_512 | 0.769 (0.714–0.821) | 0.764 (0.705–0.818) |   -0.005 |       257 |                      189 |
| A2_densenet_ft_focal_224       | 0.640 (0.582–0.699) | 0.645 (0.577–0.710) |    0.004 |       257 |                      189 |
| A3_efficientnetb0_ft_focal_512 | 0.711 (0.654–0.768) | 0.728 (0.667–0.786) |    0.017 |       257 |                      189 |
| A4_resnet50v2_ft_focal_512     | 0.751 (0.694–0.806) | 0.764 (0.703–0.821) |    0.013 |       257 |                      189 |

## Grad-CAM localisation

| Model                                     | Pointing-game hit % (95% CI)   |   Hit % (malignant only) |   Chance % (whole image) |   Chance % (within breast) |   Concentration (median) |   Energy outside breast % |
|:------------------------------------------|:-------------------------------|-------------------------:|-------------------------:|---------------------------:|-------------------------:|--------------------------:|
| B0_densenet_frozen_bce_224                | 2.8 (1.5–4.5)                  |                      4.7 |                      2.4 |                        6.4 |                     0.83 |                      63.7 |
| M1_custom_cnn_focal_512                   | 5.7 (3.3–8.1)                  |                      8.8 |                      2.4 |                        6.4 |                     1.53 |                      39.8 |
| M2_densenet_ft_bce_512                    | 10.5 (7.2–14.0)                |                     19.7 |                      2.4 |                        6.4 |                     0.8  |                      62.5 |
| M3_densenet_ft_focal_512                  | 11.4 (8.0–14.9)                |                     20.7 |                      2.4 |                        6.4 |                     0.6  |                      59.7 |
| A1_densenet_ft_focal_noaug_512            | 5.2 (2.7–7.9)                  |                      8.8 |                      2.4 |                        6.4 |                     0.61 |                      59.9 |
| A2_densenet_ft_focal_224                  | 2.6 (1.1–4.3)                  |                      5.2 |                      2.4 |                        6.4 |                     0.04 |                      63.6 |
| A3_efficientnetb0_ft_focal_512            | 16.2 (12.5–20.1)               |                     25.4 |                      2.4 |                        6.4 |                     1.59 |                      48.9 |
| A4_resnet50v2_ft_focal_512                | 15.7 (12.2–19.2)               |                     26.4 |                      2.4 |                        6.4 |                     1.06 |                      57.9 |
| M3_densenet_ft_focal_512 (random weights) | 4.8 (2.9–6.9)                  |                      7.3 |                      2.4 |                        6.4 |                     0    |                      29.9 |
