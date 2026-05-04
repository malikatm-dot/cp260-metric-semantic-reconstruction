# CP260 Robotic Perception — Final Project
## Metric-Semantic Pose Estimation of PC Back-Panel Components

**Malika T M** (SR No. 25909) · **Yashita Jaiswal**  
Robert Bosch Centre for Cyber-Physical Systems (RBCCPS)  
Indian Institute of Science, Bangalore · May 2026

---

## Overview

This project estimates the 6-DoF pose of physical components on the back panel of a desktop PC from 16 posed RGB images. Each pose is represented as an **Oriented Bounding Box (OBB)** — a 3D center, half-extents, and a rotation matrix.

**Baseline parts:**
- Power socket (IEC C14 inlet)
- Ethernet socket (RJ-45 port)
- VGA socket

**Bonus:** Any new part specified at evaluation time using a text prompt (via Grounding DINO).

### Validation (VGA socket vs ground truth)
| Metric | Result |
|--------|--------|
| Center distance | **0.6 mm** |
| Extent error | **0.3 mm** |
| Rotation error | **6.8°** |

---

## Pipeline

```
16 RGB Images + poses.json + intrinsics
        │
        ▼
Manual 2D bounding box annotations (frames 471, 496)
        │
        ▼
Multi-view triangulation (dense grid, 400 pts/ROI)
  - OpenCV triangulatePoints
  - Reprojection error filter (<50px)
  - MAD outlier rejection
        │
        ▼
OBB fitting
  - SVD plane normal estimation
  - World-X axis alignment for panel horizontal
  - 6mm physical depth prior
        │
        ▼
answers.json  (center, extent, rotation per entity)
```

For the **bonus evaluation**, Grounding DINO replaces manual annotation:
```
Text prompt → Grounding DINO → top-2 detections → same triangulation + OBB pipeline
```

---

## Repository Structure

```
cp260-metric-semantic-reconstruction/
├── README.md
├── src/
│   ├── pose_estimation.py     # triangulation + OBB fitting
│   ├── semantic.py            # Grounding DINO bonus eval
│   ├── data_loader.py         # image + pose loading
│   ├── config.py              # intrinsics, annotations, paths
│   └── utils.py               # projection, MAD outlier rejection
├── docs/
│   ├── CP260_Report.pdf       # full project report
│   └── instructions.md        # detailed run instructions
└── output/
    └── answers.json           # final OBB pose estimates
```

---

## Requirements

```
python >= 3.10
opencv-python-headless
numpy
scipy
torch
transformers
accelerate
open3d
```

Install with:
```bash
pip install opencv-python-headless numpy scipy torch transformers accelerate open3d
```

---

## Running the Code

### Option 1: Google Colab (recommended)
Upload `CP260_Final_Colab.ipynb` to [Google Colab](https://colab.research.google.com), enable a T4 GPU, and run all cells in order.

### Option 2: Local / command line

**1. Clone the repo**
```bash
git clone https://github.com/malikatm-dot/cp260-metric-semantic-reconstruction
cd cp260-metric-semantic-reconstruction
```

**2. Set up dataset**

Place the dataset in a `Data/` folder at the repo root:
```
Data/
├── frame_000319.png
├── frame_000333.png
├── ...
├── frame_000531.png
└── poses.json
```

**3. Update config**

Edit `src/config.py` to set your dataset path:
```python
DATA_DIR = 'Data'   # path to your Data folder
```

**4. Run baseline pose estimation**
```bash
python src/pose_estimation.py
```
Output saved to `output/answers.json`.

**5. Run bonus eval (new part)**
```bash
python src/semantic.py --part "USB port . USB connector" --name usb_port
```

---

## Camera Intrinsics

Calibrated intrinsics from `intrinsic.json`:

| Parameter | Value |
|-----------|-------|
| fx | 1477.010 |
| fy | 1480.442 |
| cx | 1298.250 |
| cy | 686.820 |
| Image size | 2560 × 1440 |

> **Note:** COLMAP auto-estimation returned fx=fy=3072 (2× error). Always use the calibrated values.

---

## Manual Annotations

2D bounding boxes (pixel coordinates, 2560×1440 resolution):

| Entity | Frame | x1 | y1 | x2 | y2 |
|--------|-------|-----|-----|-----|-----|
| power_socket | 471 | 1140 | 1020 | 1280 | 1100 |
| power_socket | 496 | 1670 | 1060 | 1820 | 1155 |
| ethernet_socket | 471 | 1160 | 305 | 1280 | 405 |
| ethernet_socket | 496 | 1735 | 305 | 1845 | 405 |
| vga_socket | 471 | 1101 | 357 | 1285 | 423 |
| vga_socket | 496 | 1671 | 364 | 1855 | 430 |

---

## Output Format

`answers.json` follows this structure:

```json
[
  {
    "entity": "power_socket",
    "obb": {
      "center": [x, y, z],
      "extent": [w, h, d],
      "rotation": [[r00, r01, r02],
                   [r10, r11, r12],
                   [r20, r21, r22]]
    }
  },
  ...
]
```

---

## References

- Hartley & Zisserman, *Multiple View Geometry in Computer Vision*, 2004
- Liu et al., *Grounding DINO*, arXiv:2303.05499, 2023
- Schonberger & Frahm, *Structure-from-Motion Revisited*, CVPR 2016
