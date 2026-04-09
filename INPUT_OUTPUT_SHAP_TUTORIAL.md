# 🎯 Complete Tutorial: Input → Model Output → SHAP Explanation

This guide shows **exactly how** to:
1. ✅ Give an image as input
2. ✅ Get model predictions (output)
3. ✅ Generate SHAP explanations

---

## 📋 Table of Contents
1. [Quick Start (Copy-Paste Ready)](#quick-start)
2. [Method 1: Using Main.py (Recommended)](#method-1-using-mainpy-recommended)
3. [Method 2: Using inference.py + SHAP](#method-2-using-inferencepy--shap)
4. [Method 3: Python Script (Advanced)](#method-3-python-script-advanced)
5. [Understanding the Output](#understanding-the-output)

---

## 🚀 Quick Start

### **Fastest Way: Single Command**

```bash
cd /Users/nihaldastagiri/Desktop/XAI
source xai-env/bin/activate

# Activate environment (if not already)
python main.py explain --config configs/config.yaml \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

**Result**: You get:
- Model prediction with confidence
- SHAP heatmap showing important regions
- Visualizations saved to `experiments/results/`

---

## 📖 Method 1: Using main.py (Recommended)

### **Step 1: Prepare Your Input Image**

Choose a test image from:
```
data/raw/MRI/Testing/
├── glioma/        (402 images)
├── meningioma/    (402 images)
├── notumor/       (402 images)
└── pituitary/     (402 images)
```

Example image:
```bash
data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

### **Step 2: Run the Pipeline**

```bash
python main.py explain \
  --config configs/config.yaml \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

### **Step 3: View Output**

Results are saved to:
```
experiments/results/
├── shap/
│   ├── Te-gl_1_original.jpg        # Original MRI scan
│   ├── Te-gl_1_shap_heatmap.jpg    # SHAP heatmap
│   ├── Te-gl_1_overlay.jpg         # Heatmap overlay on original
│   ├── Te-gl_1_predictions.json    # Model predictions JSON
│   └── Te-gl_1_summary.txt         # Text summary
└── logs/
    └── explain.log                 # Detailed log
```

### **Console Output Example**

```
2026-04-09 14:23:45 - INFO - Loading model: models/checkpoints/best_model.pth
2026-04-09 14:23:46 - INFO - Model loaded successfully
2026-04-09 14:23:47 - INFO - Loading image: data/raw/MRI/Testing/glioma/Te-gl_1.jpg
2026-04-09 14:23:48 - INFO - Generating model predictions...

==================================================
MODEL PREDICTIONS
==================================================
Predicted Class: glioma
Confidence: 95.23%
Class Probabilities:
  - glioma     : 0.9523 (95.23%) ← HIGHEST
  - meningioma : 0.0362 (3.62%)
  - notumor    : 0.0089 (0.89%)
  - pituitary  : 0.0026 (0.26%)

==================================================
GENERATING SHAP EXPLANATION
==================================================
2026-04-09 14:24:15 - INFO - Preparing background images...
2026-04-09 14:24:45 - INFO - Computing SHAP values...
2026-04-09 14:25:30 - INFO - Generating visualizations...

==================================================
SHAP EXPLANATION COMPLETE
==================================================
Output files:
  - Original image: experiments/results/shap/Te-gl_1_original.jpg
  - SHAP heatmap: experiments/results/shap/Te-gl_1_shap_heatmap.jpg
  - Overlay: experiments/results/shap/Te-gl_1_overlay.jpg
  - Predictions JSON: experiments/results/shap/Te-gl_1_predictions.json
  - Summary: experiments/results/shap/Te-gl_1_summary.txt
```

---

## 🔄 Method 2: Using inference.py + SHAP

### **Step 1: Get Model Predictions Only (Fast)**

```bash
python inference.py \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

**Output**:
```
Predicted Class: glioma (index: 0)
Confidence: 95.23%

Class Probabilities:
  glioma     : 0.9523 (95.23%) ← HIGHEST
  meningioma : 0.0362 (3.62%) 
  notumor    : 0.0089 (0.89%)
  pituitary  : 0.0026 (0.26%)

Raw Logits: [3.2145, -0.8523, -2.1034, -1.9854]
```

Processing time: **~0.3 seconds** ⚡

### **Step 2: Generate SHAP for Top Predictions**

If you want SHAP only for the top predictions (faster than Method 1):

```bash
python main.py explain \
  --config configs/config.yaml \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg \
  --top_k 1
```

This generates SHAP explanation only for the top-1 predicted class (faster).

---

## 💻 Method 3: Python Script (Advanced)

### **Complete Python Script**

Create a file `my_inference_script.py`:

```python
"""
Complete example: Input → Model Output → SHAP Explanation
"""

import torch
import logging
from pathlib import Path

# Setup
from models.cnn_model import load_model
from utils.preprocessing import preprocess_image
from utils.mri_dataset_loader import MRIDataLoader
from utils.visualization import overlay_heatmap, plot_predictions
from xai.shap_explainer import SHAPExplainer

# Setup logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# ============================================================
# STEP 1: DEFINE INPUTS
# ============================================================
IMAGE_PATH = "data/raw/MRI/Testing/glioma/Te-gl_1.jpg"
MODEL_PATH = "models/checkpoints/best_model.pth"
OUTPUT_DIR = "experiments/results/my_test/"
CLASS_NAMES = ["glioma", "meningioma", "notumor", "pituitary"]
DEVICE = 'cuda' if torch.cuda.is_available() else 'cpu'

# Create output directory
Path(OUTPUT_DIR).mkdir(parents=True, exist_ok=True)

logger.info("="*60)
logger.info("STARTING INFERENCE + SHAP PIPELINE")
logger.info("="*60)

# ============================================================
# STEP 2: LOAD MODEL
# ============================================================
logger.info(f"Loading model from: {MODEL_PATH}")
model = load_model(MODEL_PATH, device=DEVICE)
model.eval()
logger.info("✓ Model loaded successfully")

# ============================================================
# STEP 3: LOAD AND PREPROCESS IMAGE
# ============================================================
logger.info(f"Loading image from: {IMAGE_PATH}")
image_tensor = preprocess_image(IMAGE_PATH, device=DEVICE)
logger.info(f"✓ Image shape: {image_tensor.shape}")

# ============================================================
# STEP 4: GET MODEL PREDICTIONS
# ============================================================
logger.info("Generating model predictions...")

with torch.no_grad():
    logits = model(image_tensor.unsqueeze(0))
    probabilities = torch.softmax(logits, dim=1)

# Extract results
pred_class_idx = probabilities.argmax(dim=1).item()
pred_class_name = CLASS_NAMES[pred_class_idx]
confidence = probabilities[0, pred_class_idx].item()
all_probs = probabilities[0].detach().cpu().numpy()

logger.info("="*60)
logger.info("MODEL PREDICTIONS")
logger.info("="*60)
logger.info(f"Predicted Class: {pred_class_name}")
logger.info(f"Confidence: {confidence:.2%}")
logger.info(f"Logits: {logits[0].detach().cpu().numpy()}")
logger.info("\nClass Probabilities:")
for i, (class_name, prob) in enumerate(zip(CLASS_NAMES, all_probs)):
    marker = " ← HIGHEST" if i == pred_class_idx else ""
    logger.info(f"  {class_name:12s}: {prob:.4f} ({prob*100:.2f}%){marker}")

# ============================================================
# STEP 5: GENERATE SHAP EXPLANATION
# ============================================================
logger.info("\n" + "="*60)
logger.info("GENERATING SHAP EXPLANATION")
logger.info("="*60)

# Load background data for SHAP
logger.info("Loading background images for SHAP...")
data_loader = MRIDataLoader(
    dataset_dir="data/raw/MRI",
    image_size=224,
    batch_size=32
)
background_loader = data_loader.get_background_loader()
logger.info("✓ Background loaded")

# Create SHAP explainer
logger.info("Initializing SHAP explainer...")
explainer = SHAPExplainer(
    background_loader=background_loader,
    num_samples=100,
    device=DEVICE,
    batch_size=32
)
logger.info("✓ SHAP explainer ready")

# Compute SHAP values
logger.info("Computing SHAP values (this may take 2-5 minutes)...")
shap_values = explainer.explain(
    image_tensor=image_tensor,
    model=model,
    predicted_class=pred_class_idx,
    class_names=CLASS_NAMES
)
logger.info("✓ SHAP values computed")

# ============================================================
# STEP 6: VISUALIZE AND SAVE
# ============================================================
logger.info("\nGenerating visualizations...")

# Save SHAP heatmap
heatmap_path = f"{OUTPUT_DIR}/{Path(IMAGE_PATH).stem}_shap_heatmap.jpg"
explainer.visualize_shap(
    image_tensor=image_tensor,
    shap_values=shap_values,
    class_name=pred_class_name,
    output_path=heatmap_path
)
logger.info(f"✓ SHAP heatmap saved: {heatmap_path}")

# Save overlay
overlay_path = f"{OUTPUT_DIR}/{Path(IMAGE_PATH).stem}_overlay.jpg"
overlay_heatmap(
    image_path=IMAGE_PATH,
    shap_heatmap_path=heatmap_path,
    output_path=overlay_path
)
logger.info(f"✓ Overlay saved: {overlay_path}")

# Save predictions summary
summary_path = f"{OUTPUT_DIR}/{Path(IMAGE_PATH).stem}_summary.txt"
with open(summary_path, 'w') as f:
    f.write("="*60 + "\n")
    f.write("INFERENCE + SHAP SUMMARY\n")
    f.write("="*60 + "\n\n")
    f.write(f"Image: {IMAGE_PATH}\n")
    f.write(f"Model: {MODEL_PATH}\n\n")
    f.write("MODEL PREDICTIONS:\n")
    f.write(f"  Predicted Class: {pred_class_name}\n")
    f.write(f"  Confidence: {confidence:.2%}\n\n")
    f.write("Class Probabilities:\n")
    for class_name, prob in zip(CLASS_NAMES, all_probs):
        f.write(f"  {class_name:12s}: {prob:.4f} ({prob*100:.2f}%)\n")
    f.write("\nOutput Files:\n")
    f.write(f"  SHAP Heatmap: {heatmap_path}\n")
    f.write(f"  Overlay: {overlay_path}\n")
logger.info(f"✓ Summary saved: {summary_path}")

logger.info("\n" + "="*60)
logger.info("PIPELINE COMPLETE!")
logger.info("="*60)
logger.info(f"\nOutput directory: {OUTPUT_DIR}")
logger.info(f"View files:")
logger.info(f"  - {heatmap_path}")
logger.info(f"  - {overlay_path}")
logger.info(f"  - {summary_path}")
```

### **Run the Script**

```bash
python my_inference_script.py
```

---

## 📊 Understanding the Output

### **1. Model Predictions**

```
Predicted Class: glioma
Confidence: 95.23%
```

**What it means:**
- **Predicted Class**: The model's best guess (highest probability)
- **Confidence**: How sure the model is (0-100%)
  - **95%+**: Very confident ✅
  - **70-94%**: Confident ⚠️
  - **50-69%**: Uncertain ⚠️⚠️
  - **<50%**: Very uncertain ❌

### **2. Class Probabilities**

```
  glioma     : 0.9523 (95.23%) ← HIGHEST
  meningioma : 0.0362 (3.62%)
  notumor    : 0.0089 (0.89%)
  pituitary  : 0.0026 (0.26%)
```

**What it means:**
- Model's distribution across all classes
- Sum always equals 1.0 (100%)
- Higher = more likely

### **3. SHAP Explanation (Heatmap)**

**Red regions** = Push toward predicted class
- **Bright red**: Strong contribution
- **Light red**: Weak contribution

**Blue regions** = Push away from predicted class
- **Bright blue**: Strong negative contribution
- **Light blue**: Weak negative contribution

**Gray regions** = No significant contribution

### **4. Example Interpretation**

```
Image: MRI scan with brain tumor
Model says: "This is GLIOMA with 95% confidence"
SHAP shows: Red heatmap in center region

Interpretation:
  "The model focused on the CENTER REGION of the brain
   and found features that match GLIOMA patterns."
```

---

## 🎯 Common Use Cases

### **Use Case 1: Single Image Prediction + Explanation**

```bash
python main.py explain \
  --config configs/config.yaml \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

### **Use Case 2: Batch Process Multiple Images**

```bash
# Create a loop
for img in data/raw/MRI/Testing/glioma/*.jpg; do
  echo "Processing: $img"
  python main.py explain \
    --config configs/config.yaml \
    --model_path models/checkpoints/best_model.pth \
    --image_path "$img" &
done
```

### **Use Case 3: Fast Prediction Only (No SHAP)**

```bash
# For just predictions (no explanation)
python inference.py \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

**Time:** ~0.3 seconds vs ~3-5 minutes for SHAP

### **Use Case 4: Python Integration**

```python
# Use in your own code
from models.cnn_model import load_model
from utils.preprocessing import preprocess_image

model = load_model("models/checkpoints/best_model.pth")
image = preprocess_image("path/to/image.jpg")

with torch.no_grad():
    output = model(image.unsqueeze(0))
    probs = torch.softmax(output, dim=1)
    
print(f"Predicted: {probs.argmax().item()}")
print(f"Confidence: {probs.max().item():.2%}")
```

---

## ⏱️ Performance Guide

| Task | Time | Command |
|------|------|---------|
| **Model Prediction Only** | ~0.3s | `python inference.py ...` |
| **SHAP Explanation** | ~3-5 min | `python main.py explain ...` |
| **Batch (10 images)** | ~30-50 min | Loop + `main.py explain` |
| **Batch Predictions (10 images)** | ~3s | Loop + `inference.py` |

---

## 🐛 Troubleshooting

### **Error: "FileNotFoundError: model not found"**

```bash
# Solution: Train the model first
python main.py train --config configs/config.yaml
```

### **Error: "No such file or directory: image.jpg"**

```bash
# Solution: Check image path exists
ls -la data/raw/MRI/Testing/glioma/

# Use correct path
python main.py explain --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

### **Error: "CUDA out of memory"**

```bash
# Solution: Use CPU instead
# The code auto-detects, but if you have issues:
# Edit config.yaml and set device: 'cpu'

python main.py explain --config configs/config.yaml ...
# (Script will automatically use CPU)
```

### **SHAP taking too long?**

```bash
# Solution: Reduce background samples (fewer = faster but less accurate)
# Edit in Python script:
explainer = SHAPExplainer(
    background_loader=background_loader,
    num_samples=50,  # Instead of 100
    device=DEVICE
)
```

---

## ✅ Checklist: Are You Ready?

- [x] Model trained: `models/checkpoints/best_model.pth` exists
- [x] Test images available: `data/raw/MRI/Testing/` has images
- [x] Config file ready: `configs/config.yaml` exists
- [x] Python environment activated: `source xai-env/bin/activate`
- [x] All dependencies installed: Check with `pip list | grep torch`

---

## 🚀 Start Now!

### **Simplest Command (Copy-Paste Ready)**

```bash
cd /Users/nihaldastagiri/Desktop/XAI
source xai-env/bin/activate

python main.py explain \
  --config configs/config.yaml \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

Then check results in: `experiments/results/`

---

## 📚 Next Steps

1. **Try all 3 methods** to find what works best for you
2. **Experiment with different images** from different tumor types
3. **Compare predictions vs ground truth** to understand model accuracy
4. **Analyze SHAP heatmaps** to understand model reasoning
5. **Save results** for your project report

---

**Questions?** Check the logs:
```bash
tail -f logs/main.log
```

Happy inferencing! 🎯

