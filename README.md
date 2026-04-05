# Maritime Small Object Detection Dataset
### 7-Class UAV Maritime Dataset for Search, Rescue & Safety Monitoring

[![Kaggle](https://img.shields.io/badge/Dataset-Kaggle-20BEFF?style=flat&logo=kaggle)](https://www.kaggle.com/datasets/codderboy/final-mixed-dataset-prepare-7-class-v2)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![YOLO Format](https://img.shields.io/badge/Format-YOLOv12-blue)](https://github.com/sunsmarterjie/yolov12)
[![Classes](https://img.shields.io/badge/Classes-7-green)]()
[![Train Images](https://img.shields.io/badge/Train%20Images-44%2C940-orange)]()

---

> **Associated Paper:**
> *Explainable and Generalisable Deep Learning for Cross-Dataset Maritime Search, Rescue, and Safety Monitoring Using UAVs*

---

## Overview

This repository provides dataset documentation, metadata, and download instructions for the **Mixed Maritime 7-Class Detection Dataset** - a preprocessed, YOLO-format aerial dataset designed for small-object detection in maritime search and rescue (SAR) and safety monitoring scenarios using UAVs.

The dataset was constructed by merging multiple publicly available maritime surveillance datasets, then subjected to a targeted preprocessing pipeline to address class imbalance, small-object invisibility, and a high rate of empty annotations.

---

## Dataset Summary

| Property | Value |
|---|---|
| **Task** | Object Detection |
| **Format** | YOLO (`.txt` labels, normalized xywh) |
| **Input Resolution** | 640 × 640 px |
| **Number of Classes** | 7 |
| **Total Train Images** | 44,940 |
| **Total Val Images** | 1,098 |
| **Total Test Images** | 558 |
| **Total Annotations (Train)** | 74,535 |
| **Empty Labels** | 0 (removed during preprocessing) |
| **Image Format** | JPEG |
| **Hosted On** | Kaggle |

---

## Classes

| ID | Class | Train Instances | Notes |
|---|---|---|---|
| 0 | `drowning` | 1,366 | High-priority safety class |
| 1 | `swimming` | 7,837 | Augmented (2,148 → 7,837) |
| 2 | `life_buoy` | 873 | Safety equipment |
| 3 | `boat` | 17,525 | Vessel class |
| 4 | `buoy` | 5,742 | Small object; SAHI tiled |
| 5 | `jetski` | 4,963 | Small object; SAHI tiled |
| 6 | `swimmer` | 36,229 | Small object; SAHI tiled |

---

## Preprocessing Pipeline

The raw merged dataset had significant quality issues that were addressed with a 3-stage pipeline:

### Stage 1 — Remove Empty Labels
- **Problem:** 18,901 out of 38,234 training labels (49%) were empty
- **Fix:** All empty label files and their corresponding images were removed
- **Result:** 19,333 clean image-label pairs retained

### Stage 2 — SAHI-Style Tiling (3× Magnification)
- **Problem:** `buoy`, `jetski`, and `swimmer` had median object sizes of 10 px, 26 px, and 28 px respectively — effectively invisible at 640×640 training resolution
- **Fix:** Images containing small instances were tiled into 208×208 crops and resized to 640×640 (3.1× magnification)
- **Result:** Median sizes increased to 30 px, 72 px, and 77 px respectively; 22,133 tile images added

| Class | Before (px) | After tiling (px) | Improvement |
|---|---|---|---|
| buoy | 10 | 30 | 3.0× |
| jetski | 26 | 72 | 2.8× |
| swimmer | 28 | 77 | 2.8× |

### Stage 3 — Swimming Class Augmentation
- **Problem:** Only 2,148 `swimming` instances; low recall (R = 0.704) in baseline experiments
- **Fix:** 8 augmentation types applied (horizontal/vertical flip, brightness, contrast, saturation jitter, blur combinations)
- **Result:** 2,148 → 7,837 instances; 3,474 augmented images added

### Final Training Split Composition

| Type | Images |
|---|---|
| Original (non-empty) | 19,333 |
| SAHI tiles | 22,133 |
| Swimming augmentations | 3,474 |
| **Total** | **44,940** |

---

## Download

The dataset is hosted on Kaggle and available for free download:

**[Download on Kaggle →](https://www.kaggle.com/datasets/codderboy/final-mixed-dataset-prepare-7-class-v2)**

### Using the Kaggle API

```bash
# Install Kaggle CLI
pip install kaggle

# Download dataset
kaggle datasets download -d codderboy/final-mixed-dataset-prepare-7-class-v2

# Unzip
unzip final-mixed-dataset-prepare-7-class-v2.zip -d maritime_dataset
```

### Directory Structure

After downloading and unzipping, the dataset follows standard YOLO directory layout:

```
maritime_dataset/
├── train/
│   ├── images/          # 44,940 JPEG images
│   └── labels/          # 44,940 YOLO .txt label files
├── valid/
│   ├── images/          # 1,098 images
│   └── labels/          # 1,098 label files
├── test/
│   ├── images/          # 558 images
│   └── labels/          # 558 label files
└── data.yaml            # Dataset config for YOLO training
```

### data.yaml

```yaml
path: /path/to/maritime_dataset
train: train/images
val: valid/images
test: test/images

nc: 7
names: ['drowning', 'swimming', 'life_buoy', 'boat', 'buoy', 'jetski', 'swimmer']
```

---

## Training Example

```python
from ultralytics import YOLO

model = YOLO('yolov8n.pt')  # or yolov12s, yolov12m, etc.

model.train(
    data='maritime_dataset/data.yaml',
    epochs=10,
    imgsz=640,
    batch=16,
    project='maritime_sar',
    name='yolov8n_run1'
)
```

---

## Cross-Dataset Evaluation

The model trained on this dataset was evaluated zero-shot on two independent maritime benchmarks:

| Dataset | Classes Matched | AP@0.5 (Overall) |
|---|---|---|
| [MOBDrone](https://github.com/grains-polito/MOBDrone](https://aimh.isti.cnr.it/dataset/mobdrone/)) | boat, life_buoy, person | 0.1747 |
| [SeaDronesSee-ODV2](https://github.com/Ben93kie/SeaDronesSee](https://seadronessee.cs.uni-tuebingen.de/)) | boat, buoy, jetski, swimmer | 0.0937 |

These results reflect zero-shot generalization without any fine-tuning on the target datasets.

---

## Label Format

Each `.txt` label file follows the standard YOLO format:

```
<class_id> <cx> <cy> <width> <height>
```

All values are normalized to `[0, 1]` relative to image dimensions. One object per line.

**Example:**
```
6 0.512341 0.348291 0.043210 0.055120
3 0.201842 0.671023 0.312940 0.198301
```

---

## Citation

If you use this dataset in your research, please cite:

```bibtex
@dataset{muktadir2025maritime,
  author    = {Muktadir, S. K. and others},
  title     = {Mixed Maritime 7-Class UAV Detection Dataset},
  year      = {2025},
  publisher = {Kaggle},
  url       = {https://www.kaggle.com/datasets/codderboy/final-mixed-dataset-prepare-7-class-v2}
}
```

And the associated paper:

```bibtex
@article{muktadir2025explainable,
  author  = {Muktadir, S. K. and others},
  title   = {Explainable and Generalisable Deep Learning for Cross-Dataset Maritime Search, Rescue, and Safety Monitoring Using UAVs},
  journal = {},
  year    = {2025}
}
```

---

## License

This dataset is released under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license.
You are free to share and adapt the material for any purpose, provided appropriate credit is given.

---

## Contact

For questions, please open a [GitHub Issue](https://github.com/skmuktadir/maritime-detection-dataset/issues) or reach out via Kaggle.
