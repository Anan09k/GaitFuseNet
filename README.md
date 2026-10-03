# GaitFuseNet

### A Cross-Modal Attention-Fusion Network for Wearable-Sensor-Based Parkinson's Disease Gait Assessment

GaitFuseNet is a deep learning framework for Parkinson's disease gait
assessment using synchronized wearable inertial measurement unit (IMU)
and plantar-pressure/insole signals.

## Overview

The project investigates whether combining multiple wearable sensing
modalities can improve Parkinson's disease gait assessment.

The framework uses:

- Wearable IMU signals
- Plantar-pressure/insole signals
- CNN-based temporal feature extraction
- Bidirectional LSTM
- Cross-modal attention
- Gated multimodal fusion
- Temporal self-attention

## Dataset

Experiments are conducted using the WearGait-PD dataset.

The dataset contains synchronized wearable sensor recordings from
participants with Parkinson's disease and healthy controls.

The dataset itself is not included in this repository.

## Data

### IMU

13 wearable IMU locations:

- LowerBack
- R_Wrist
- L_Wrist
- R_MidLatThigh
- L_MidLatThigh
- R_LatShank
- L_LatShank
- R_DorsalFoot
- L_DorsalFoot
- R_Ankle
- L_Ankle
- Xiphoid
- Forehead

The IMU data contains 286 features.

### Insole

The plantar-pressure system contains 50 raw features,
which are further processed into different feature representations
for experimentation.

## Preprocessing

The preprocessing pipeline includes:

- Missing-value handling
- Linear interpolation of short missing segments
- Removal of incomplete recordings
- Temporal windowing
- Feature normalization
- Sensor-noise augmentation
- Savitzky-Golay smoothing
- Dynamic plantar-pressure feature extraction

## Experimental Setup

Experiments use subject-level 5-fold cross-validation to prevent
samples from the same participant from appearing in both training
and evaluation folds.

Random seed:

    42

Window size:

    300 samples

Sampling frequency:

    100 Hz

Window duration:

    approximately 3 seconds

## Model Architecture

The GaitFuseNet architecture consists of:

    IMU ──> CNN ──> BiLSTM ──┐
                             │
                             ├─> Cross-Modal Attention
                             │
    Insole -> CNN -> BiLSTM ─┘
                             │
                         Gated Fusion
                             │
                     Temporal Attention
                             │
                     Classification
                     Regression

## Evaluation Metrics

Classification:

- Accuracy
- Precision
- Recall
- F1-score
- Window-level AUROC
- Subject-level AUROC

Regression:

- MAE
- RMSE

## Experiments

The repository contains experiments investigating:

1. IMU-only models
2. Insole-only models
3. Naive multimodal fusion
4. Cross-modal attention fusion
5. Gated fusion
6. Reduced-sensor IMU models
7. Different insole representations
8. Insole preprocessing
9. 5-fold subject-level cross-validation

## Results

Results will be added after completion of the experiments.

## Repository Structure

    GaitFuseNet/
    ├── data/
    ├── notebooks/
    ├── results/
    ├── README.md
    ├── requirements.txt
    └── .gitignore

## Requirements

Python 3.x

Major dependencies:

- PyTorch
- NumPy
- Pandas
- SciPy
- scikit-learn
- Matplotlib
- Seaborn

Install dependencies using:

    pip install -r requirements.txt

## Reproducibility

Experiments use a fixed random seed of 42 and subject-level
cross-validation.

## License

To be added.
