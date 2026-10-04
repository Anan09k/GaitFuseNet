# GaitFuseNet

### An Attention-Based Multimodal Deep Learning Framework for Wearable-Sensor-Based Parkinson's Disease Gait Assessment

GaitFuseNet is a multimodal deep learning framework for Parkinson's disease (PD) gait assessment using synchronized wearable inertial measurement unit (IMU) and plantar-pressure/insole signals. The project investigates whether cross-modal attention-based fusion of these two sensing modalities improves PD classification and severity estimation beyond what IMU data alone achieves.

---

## Key Finding

Across a complete ablation chain — architecture selection, insole feature engineering, insole signal preprocessing, and rigorous 5-fold subject-grouped cross-validation — **IMU-only models consistently matched or outperformed every multimodal fusion configuration tested.** This holds across naive concatenation, cross-modal attention, and gated fusion, across three insole feature representations, and across four insole signal-preprocessing strategies.

| Configuration | Subject-Level AUROC (5-fold) |
|---|---:|
| **IMU-only** | **0.9655 ± 0.0174** |
| Multimodal, Butterworth-filtered insole (best fusion result) | 0.9620 ± 0.0229 |
| Multimodal, median-filtered insole | 0.9557 ± 0.0360 |
| Multimodal, unfiltered / Savitzky-Golay insole | 0.9493 ± 0.03 |

This project's primary contribution is therefore a rigorously validated negative result, accompanied by a systematic investigation of which factors (insole feature engineering, signal filtering, IMU sensor coverage) measurably affect the gap between fusion and unimodal performance — even where they do not close it.

---

## Overview

Parkinson's disease causes characteristic changes in gait and motor function. Wearable sensors provide an opportunity to capture these changes objectively through body-motion and plantar-pressure measurements.

GaitFuseNet processes IMU and insole signals through separate modality-specific temporal encoders and models their interactions using bidirectional cross-modal attention, followed by gated multimodal fusion and temporal self-attention. The framework performs two simultaneous prediction tasks:

1. **PD classification** — Parkinson's disease vs. healthy control
2. **Severity regression** — a continuous motor-severity score derived from MDS-UPDRS Part III

---

## Dataset

Experiments use the **WearGait-PD** dataset — an open-access multimodal wearable-sensor dataset containing synchronized recordings from individuals with Parkinson's disease and age-matched controls.

The original dataset contains:

- **185 participants** (100 with Parkinson's disease, 85 age-matched controls)
- **13 body-worn IMUs** per participant
- **Sensorized insoles** on both feet
- Synchronized recordings sampled at **100 Hz**
- Clinical data including item-level MDS-UPDRS scores

The dataset itself is **not included in this repository**; see `data/README.md` for access instructions.

After removing structurally incomplete recordings and excluding non-gait tasks, the experimental dataset used in this project contained:

- **169 participants**
- **24,565 windows**, each 300 samples (3 seconds) long, with 150-sample (50%) overlap

---

## Sensor Modalities

### IMU

Thirteen IMU locations: LowerBack, R_Wrist, L_Wrist, R_MidLatThigh, L_MidLatThigh, R_LatShank, L_LatShank, R_DorsalFoot, L_DorsalFoot, R_Ankle, L_Ankle, Xiphoid, Forehead.

Each sensor provides 22 channels (acceleration, gravity-compensated acceleration, gyroscopic rotation, magnetic field, velocity increment, orientation quaternion, roll/pitch/yaw), yielding **286 IMU features** per timestep.

### Plantar-Pressure / Insole

Raw insole data contains **50 features** per timestep (16 pressure sensors, insole-mounted IMU, total force, and center-of-pressure per foot). Three derived representations were investigated; the final multimodal experiments use a **33-feature dynamic representation**:

- Regional plantar pressure (heel, midfoot, metatarsal, toe)
- Total vertical force
- Center-of-pressure (CoP) X/Y
- CoP velocity X/Y, CoP speed, CoP acceleration
- Left-foot, right-foot, and left-right asymmetry streams

---

## Preprocessing Pipeline

- Removal of structurally incomplete recordings (missing sensor channels)
- Trimming of initial "Standing" phase per trial
- Linear interpolation of short missing-value gaps (≤0.3 s), with residual-NaN windows excluded
- Participant-level data organization and temporal windowing
- Per-window (temporal) normalization of insole features; global training-set normalization for IMU
- Fold-specific normalization statistics (recomputed per cross-validation fold)
- Training-time Gaussian noise augmentation

**Insole signal-preprocessing conditions evaluated** (applied to insole signals only; IMU was not filtered):

- No filtering
- Savitzky-Golay smoothing (window 11, order 2)
- Butterworth low-pass filtering (4th order, 5 Hz cutoff)
- Median filtering

---

## Experimental Design

Experiments were conducted progressively:

1. Unimodal baselines (IMU-only, insole-only)
2. Naive multimodal fusion (concatenation)
3. Cross-modal attention fusion
4. Architecture selection (regularization, backbone size, gated fusion — selected by validation AUROC)
5. Insole feature engineering (18 → 21 → 33 features)
6. Insole signal preprocessing (filtering comparison)
7. Sensor-reduction ablation (13 IMUs → 3 IMUs)
8. Rigorous evaluation via 5-fold subject-level cross-validation

### Architecture Selection

Four GaitFuseNet configurations were compared on a single held-out split, with selection based on **validation-set AUROC** (not test performance, to avoid selection bias):

| Configuration | Validation AUROC |
|---|---:|
| Original (concatenation) | 0.9158 |
| Regularized (dropout 0.3, weight decay 1e-3, grad clipping) | 0.8709 |
| Reduced backbone (24 CNN / 48 LSTM) | 0.9121 |
| **Gated fusion (selected)** | **0.9232** |

---

## GaitFuseNet Architecture

```text
      IMU input                         Insole input
         |                                    |
      1D CNN (x2)                         1D CNN (x2)
         |                                    |
      BiLSTM                               BiLSTM
         |                                    |
         +------------> Cross-Modal <---------+
                        Attention
                     (bidirectional,
                       4 heads)
                           |
                     Gated Fusion
                           |
                 Temporal Self-Attention
                           |
                      Mean Pooling
                           |
                    Shared Dense Layer
                      /          \
                     /            \
          Classification        Regression
           (PD vs Control)    (Severity Score)
```

**Modality-specific encoders**: two 1D convolutional layers (32 channels, kernel 5, batch norm, max pooling, dropout) followed by a bidirectional LSTM (64 hidden units per direction), producing 128-dimensional per-timestep representations.

**Cross-modal attention**: bidirectional multi-head attention (4 heads) — IMU queries attend to insole keys/values, and insole queries attend to IMU keys/values, each with residual connections and layer normalization.

**Gated fusion**: a learned sigmoid gate combines the two cross-attended representations at each timestep, replacing simple concatenation.

**Temporal self-attention**: a Transformer-encoder-style self-attention block over the fused sequence, weighting different phases of the gait cycle before pooling.

**Multi-task heads**: a shared dense layer feeding two linear output heads — binary classification (BCE loss) and severity regression (MSE loss).

---

## Evaluation Metrics

**Classification**: accuracy, precision, recall, F1-score, window-level AUROC, subject-level AUROC (predictions averaged per participant before computing AUROC).

**Regression**: mean absolute error (MAE), root mean squared error (RMSE), computed on the original MDS-UPDRS Part III scale.

---

## Results

### IMU-Only Baseline (5-Fold Cross-Validation)

| Metric | Mean ± SD |
|---|---:|
| Window AUROC | **0.9494 ± 0.0196** |
| Subject AUROC | **0.9655 ± 0.0174** |
| F1-score | 0.8988 ± 0.0119 |
| Accuracy | 0.8850 ± 0.0146 |
| MAE | 9.715 ± 2.014 |
| RMSE | 12.494 ± 2.498 |

### Insole Feature Engineering (Insole-Only, 5-Fold CV)

| Representation | Window AUROC | Subject AUROC |
|---|---:|---:|
| 18-feature (raw regional + force + CoP) | 0.5127 ± 0.0060 | 0.5665 ± 0.0663 |
| 21-feature, feature-axis normalization | 0.5397 ± 0.0352 | 0.6233 ± 0.0858 |
| 21-feature, temporal normalization | 0.5808 ± 0.0583 | 0.6425 ± 0.0955 |
| **33-feature dynamic (+ CoP kinematics)** | **0.6137 ± 0.0362** | **0.7034 ± 0.0781** |

Per-window normalization and CoP kinematic features each produced consistent improvements, though insole-only performance remained far below IMU-only even with the richest representation.

### Insole Signal Preprocessing (Full Multimodal, 33-Feature Insole, 5-Fold CV)

| Preprocessing | Window AUROC | Subject AUROC | F1 | Accuracy | MAE |
|---|---:|---:|---:|---:|---:|
| None | 0.9447 ± 0.0190 | 0.9493 ± 0.0372 | 0.8847 ± 0.0069 | 0.8705 ± 0.0100 | 9.842 ± 1.695 |
| Savitzky-Golay | 0.9445 ± 0.0141 | 0.9493 ± 0.0253 | 0.8728 ± 0.0165 | 0.8592 ± 0.0161 | 10.176 ± 1.485 |
| **Butterworth** | 0.9473 ± 0.0158 | **0.9620 ± 0.0229** | 0.8955 ± 0.0242 | 0.8782 ± 0.0306 | 9.802 ± 1.867 |
| Median | **0.9506 ± 0.0193** | 0.9557 ± 0.0360 | 0.8919 ± 0.0206 | 0.8768 ± 0.0289 | 9.941 ± 1.950 |

Butterworth filtering achieved the strongest subject-level AUROC among all tested preprocessing conditions; median filtering achieved the strongest window-level AUROC, narrowly exceeding even the IMU-only window-level result (0.9506 vs. 0.9494) — though not at the subject level, which is the more clinically meaningful aggregation for this task.

### Sensor-Reduction Ablation (Single Split)

A reduced-sensor configuration (LowerBack, right ankle, left ankle — 66 features) was tested to check whether limiting IMU coverage would increase insole's relative contribution under fusion:

| Configuration | Window AUROC | Subject AUROC | MAE |
|---|---:|---:|---:|
| **3-sensor IMU-only** | **0.963** | **0.982** | 9.87 |
| 3-sensor naive fusion | 0.958 | 0.964 | 10.94 |
| 3-sensor cross-attention fusion | 0.953 | 0.964 | 11.08 |

Even with substantially reduced IMU coverage, IMU-only outperformed both fusion variants — reinforcing that the limited contribution of the insole modality cannot be explained solely by the full IMU configuration providing excessive sensor coverage

### Main Findings

- IMU data alone provides strong, consistent discriminative signal for PD gait classification (subject AUROC 0.9655 ± 0.0174 across 5-fold CV).
- Raw insole data performs near chance level (subject AUROC 0.5665); feature engineering (anatomical regions, CoP kinematics, per-window normalization) substantially improves this to 0.7034, but it remains far below IMU-only.
- **No multimodal fusion configuration — naive, cross-attention, or gated — exceeded IMU-only's subject-level AUROC**, across any insole representation or signal-preprocessing condition tested.
- Insole preprocessing quality measurably narrows the gap between fusion and IMU-only (from 0.0162 unfiltered to 0.0035 with Butterworth filtering), indicating preprocessing — not the fusion mechanism itself — is the more significant limiting factor for insole's contribution.
- Reducing IMU sensor coverage does not change this pattern: IMU-only remains stronger than fusion even with only 3 of 13 sensors retained.

---

## Repository Structure

```text
GaitFuseNet/
|
|-- README.md
|-- requirements.txt
|-- .gitignore
|
|-- notebooks/
|   |-- GaitFuseNet_Experiments.ipynb
|   |-- GaitFuseNet_CrossValidation.ipynb
|
|-- figures/
|   |-- methodology_pipeline.png
|
|-- results/
|   |-- architecture_selection/
    |-- cross_validation/
|
|-- data/
|   |-- README.md
```

The raw WearGait-PD dataset is not included in the repository.

---

## Notebooks

### `GaitFuseNet_Experiments.ipynb`
Main experimental workflow: data preprocessing, insole feature engineering, baseline models, GaitFuseNet architecture variants, cross-modal attention, gated fusion, sensor-reduction experiments, and initial (single-split) model evaluation.

### `GaitFuseNet_CrossValidation.ipynb`
Rigorous subject-level 5-fold cross-validation experiments: IMU-only, insole-only feature comparisons, multimodal GaitFuseNet, insole signal-preprocessing comparisons, fold-wise and aggregate window-/subject-level metrics, and regression evaluation.

---

## Requirements

Python 3.x

Major dependencies: PyTorch, NumPy, Pandas, SciPy, scikit-learn, Matplotlib, Seaborn

```bash
pip install -r requirements.txt
```

---

## Reproducibility

- Random seed: **42** (applied to Python, NumPy, and PyTorch, including CUDA)
- Sampling frequency: **100 Hz**
- Window size: **300 samples** (step: 150 samples)
- Participant-level data splitting (no window-level leakage)
- 5-fold `StratifiedGroupKFold` cross-validation, grouped by participant, stratified by disease label

---

## Citation

### WearGait-PD Dataset

Anderson et al. (2026). *WearGait-PD: An Open-Access Wearables Dataset for Gait in Parkinson's Disease and Age-Matched Controls.* Scientific Data, 13, 440. DOI: `10.1038/s41597-026-06806-2`

---

## License

This repository is intended for academic and research use. A formal license will be added if required.
