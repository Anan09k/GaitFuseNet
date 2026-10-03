## Dataset

This project uses the **WearGait-PD** dataset, an open-access multimodal wearable-sensor dataset developed for studying gait and motor characteristics in people with Parkinson's disease (PD) and age-matched controls.

The dataset was introduced by **Anderson et al. (2026)** in *Scientific Data*:

> Anderson, A. J., Eguren, D., Gonzalez, M. A., et al. (2026). *WearGait-PD: An Open-Access Wearables Dataset for Gait in Parkinson’s Disease and Age-Matched Controls*. Scientific Data, 13, 440.  
> DOI: https://doi.org/10.1038/s41597-026-06806-2

**Dataset paper:** https://www.nature.com/articles/s41597-026-06806-2

### Dataset Overview

WearGait-PD contains data from **185 participants**, including:

- **100 participants with Parkinson's disease**
- **85 age-matched healthy controls**

The dataset was collected across multiple study sites using a standardized experimental protocol and sensor configuration. It combines wearable sensing with reference measurements and clinical assessments, making it suitable for investigating gait abnormalities and developing computational approaches for objective Parkinson's disease assessment.

### Wearable Sensors

The dataset includes a full-body set of **13 inertial measurement units (IMUs)** positioned at:

- Forehead
- Xiphoid process
- Lower back (L4/L5)
- Left and right wrists
- Left and right lateral thighs
- Left and right lateral shanks
- Left and right ankles
- Left and right dorsum of the feet

The IMUs provide acceleration, angular velocity, magnetic field measurements, orientation, and gravity-compensated acceleration.

The study also uses **Moticon OpenGo sensorized insoles**. Each insole contains **16 plantar-pressure sensors** along with a 3-axis accelerometer and 3-axis gyroscope, providing information about foot pressure distribution and foot movement.

### Data Acquisition

The wearable sensors and reference systems were recorded at **100 Hz**. The IMUs and sensorized insoles were synchronized with a **ProtoKinetics pressure-sensing walkway**, which served as the primary reference system for temporal synchronization. Video cameras were also used for frame-by-frame annotation of participant activities and gait-related events.

The dataset therefore provides multiple synchronized modalities:

```text
IMUs
  │
  ├── Acceleration
  ├── Angular velocity
  ├── Orientation
  └── Other sensor measurements
       │
       ├──────────────┐
       │              │
       ▼              ▼
Sensorized        Pressure
Insoles           Walkway
       │              │
       └──────┬───────┘
              ▼
        Video Annotations
              │
              ▼
      Clinical Information
```

### Clinical Information

Along with sensor data, WearGait-PD provides participant-level demographic and clinical information. This includes measures such as **MDS-UPDRS scores**, medication-related information, DBS status, and other participant characteristics. The dataset also contains annotations of gait and movement events.

### Data Used in This Project

For this study, we use the synchronized **IMU and sensorized-insole modalities** for Parkinson's disease gait assessment.

The processed data used in our experiments contain:

- **13 IMU sensors**
- **286 IMU features**
- **50 raw insole features**
- 100 Hz sampling frequency
- 300-sample temporal windows (~3 seconds)
- 50% window overlap

The 286 IMU features are obtained from the 13 body-worn IMUs, while the 50 insole features represent the two sensorized insoles.

For multimodal experiments, the raw insole representation is additionally transformed into compact anatomical and dynamic feature representations, including regional plantar-pressure features and center-of-pressure (CoP) dynamics.

### Data Availability

The WearGait-PD dataset is publicly available through **Synapse (SAGE Bionetworks)**. The complete dataset is **not included in this repository** because of its size and data-access requirements.

Researchers should obtain the dataset through its official repository and follow the access and usage requirements specified by the dataset authors.

**Official dataset repository:**  
https://www.synapse.org/Synapse:syn52540892/wiki/623751

### Citation

If you use this dataset, please cite:

```bibtex
@article{anderson2026wergaitpd,
  title={WearGait-PD: An Open-Access Wearables Dataset for Gait in Parkinson's Disease and Age-Matched Controls},
  author={Anderson, Anthony J. and Eguren, David and Gonzalez, Michael A. and others},
  journal={Scientific Data},
  volume={13},
  pages={440},
  year={2026},
  publisher={Nature},
  doi={10.1038/s41597-026-06806-2}
}
```
