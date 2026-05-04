# Running Instructions

## Quick Start (Google Colab)

1. Open [Google Colab](https://colab.research.google.com)
2. Upload `CP260_Final_Colab.ipynb`
3. Runtime → Change runtime type → **T4 GPU**
4. Upload `Data.zip` to your Google Drive
5. Run all cells in order (Steps 1–9)

The notebook handles dataset extraction, pose estimation, OBB fitting, validation, and bonus eval automatically.

---

## Bonus Evaluation (New Parts on Finals Day)

In Step 8 of the notebook, update these two lines:

```python
NEW_PART_NAME   = 'usb_port'            # ← change to requested part
NEW_PART_PROMPT = 'USB port . USB connector . USB socket'  # ← change prompt
```

Then run Steps 8 and 9 only. The new part's OBB will be appended to `answers.json`.

**Good prompt format:** use ` . ` (space-dot-space) to separate alternatives:
```
'HDMI port . HDMI socket . HDMI connector'
'audio jack . 3.5mm audio . headphone port'
'DVI port . DVI connector . DVI socket'
```

---

## Local Run

### Prerequisites
```bash
pip install opencv-python-headless numpy scipy torch transformers accelerate open3d
```

### Dataset structure
```
Data/
├── frame_000319.png  ...  frame_000531.png
└── poses.json
```

### Baseline
```bash
python src/pose_estimation.py
# Output: output/answers.json
```

### Bonus (new part)
```bash
python src/semantic.py --part "USB port . USB connector" --name usb_port
# Appends to output/answers.json
```

---

## Expected Output

`output/answers.json`:
```json
[
  {"entity": "power_socket",    "obb": {"center": [...], "extent": [...], "rotation": [[...]]}},
  {"entity": "ethernet_socket", "obb": {"center": [...], "extent": [...], "rotation": [[...]]}},
  {"entity": "vga_socket",      "obb": {"center": [...], "extent": [...], "rotation": [[...]]}}
]
```

## Validation Results (VGA socket vs ground truth)
- Center distance: **0.6 mm**
- Extent error: **0.3 mm**
- Rotation error: **6.8°**
