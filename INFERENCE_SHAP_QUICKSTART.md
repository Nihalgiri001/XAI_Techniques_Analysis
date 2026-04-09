# ⚡ Quick Start: Input → Predictions → SHAP (30 seconds)

## 🎯 TL;DR - Just Run This

```bash
cd /Users/nihaldastagiri/Desktop/XAI
source xai-env/bin/activate

python main.py explain \
  --config configs/config.yaml \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

**Result**: Check `experiments/results/` for visualizations ✅

---

## 📋 What Happens

| Step | What | Time |
|------|------|------|
| 1️⃣ **Load Image** | Reads MRI scan (224×224) | <1s |
| 2️⃣ **Get Predictions** | Model predicts class + confidence | <1s |
| 3️⃣ **Compute SHAP** | Explains which pixels matter | 2-5 min |
| 4️⃣ **Visualize** | Creates heatmaps + overlay | ~10s |

---

## 📂 Input/Output

### **Input**
```
data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

### **Output**
```
experiments/results/
└── shap/
    ├── Te-gl_1_original.jpg        # Original MRI
    ├── Te-gl_1_shap_heatmap.jpg    # SHAP heatmap
    ├── Te-gl_1_overlay.jpg         # Heatmap on original
    ├── Te-gl_1_predictions.json    # Predictions JSON
    └── Te-gl_1_summary.txt         # Text summary
```

---

## 🔥 Three Ways to Do This

### **Option 1: Full Pipeline (Recommended)**
```bash
python main.py explain \
  --config configs/config.yaml \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```
**Time**: 3-5 minutes | **Output**: Predictions + SHAP heatmap

### **Option 2: Predictions Only (Fast)**
```bash
python inference.py \
  --model_path models/checkpoints/best_model.pth \
  --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```
**Time**: 0.3 seconds | **Output**: Class probabilities

### **Option 3: Python Code**
```python
import torch
from models.cnn_model import load_model
from utils.preprocessing import preprocess_image

model = load_model("models/checkpoints/best_model.pth")
image = preprocess_image("data/raw/MRI/Testing/glioma/Te-gl_1.jpg")

with torch.no_grad():
    output = model(image.unsqueeze(0))
    probs = torch.softmax(output, dim=1)
    
print(f"Prediction: {['glioma', 'meningioma', 'notumor', 'pituitary'][probs.argmax()]}")
print(f"Confidence: {probs.max():.1%}")
```

---

## 📊 Example Output

```
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
SHAP EXPLANATION
==================================================
✓ SHAP heatmap: experiments/results/shap/Te-gl_1_shap_heatmap.jpg
✓ Overlay: experiments/results/shap/Te-gl_1_overlay.jpg

Red regions = pixels that pushed model toward GLIOMA
```

---

## 🖼️ Test Image Locations

```bash
# Pick any image:
data/raw/MRI/Testing/glioma/Te-gl_1.jpg          # Try this
data/raw/MRI/Testing/meningioma/Te-me_1.jpg      # Or this
data/raw/MRI/Testing/notumor/Te-no_1.jpg         # Or this
data/raw/MRI/Testing/pituitary/Te-pi_1.jpg       # Or this
```

---

## ✅ Single Command Copy-Paste

**Glioma**:
```bash
python main.py explain --config configs/config.yaml --model_path models/checkpoints/best_model.pth --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

**Meningioma**:
```bash
python main.py explain --config configs/config.yaml --model_path models/checkpoints/best_model.pth --image_path data/raw/MRI/Testing/meningioma/Te-me_1.jpg
```

**No Tumor**:
```bash
python main.py explain --config configs/config.yaml --model_path models/checkpoints/best_model.pth --image_path data/raw/MRI/Testing/notumor/Te-no_1.jpg
```

**Pituitary**:
```bash
python main.py explain --config configs/config.yaml --model_path models/checkpoints/best_model.pth --image_path data/raw/MRI/Testing/pituitary/Te-pi_1.jpg
```

---

## 🎯 That's It!

For detailed guide, see: [`INPUT_OUTPUT_SHAP_TUTORIAL.md`](INPUT_OUTPUT_SHAP_TUTORIAL.md)

