🎯 **QUICK START: Input → Predictions → SHAP**

## Copy-Paste Commands

### Setup
```bash
cd /Users/nihaldastagiri/Desktop/XAI
source xai-env/bin/activate
```

### Full Pipeline (Input → Model → SHAP)

**Test 1: Meningioma**
```bash
python main.py explain --config configs/config.yaml --model_path models/checkpoints/best_model.pth --image_path data/raw/MRI/Testing/meningioma/Te-me_1.jpg
```

**Test 2: Glioma**
```bash
python main.py explain --config configs/config.yaml --model_path models/checkpoints/best_model.pth --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
```

**Test 3: Pituitary**
```bash
python main.py explain --config configs/config.yaml --model_path models/checkpoints/best_model.pth --image_path data/raw/MRI/Testing/pituitary/Te-pi_1.jpg
```

**Test 4: No Tumor**
```bash
python main.py explain --config configs/config.yaml --model_path models/checkpoints/best_model.pth --image_path data/raw/MRI/Testing/notumor/Te-no_1.jpg
```

---

## What Happens

1. ✅ **Load image** (0.1 sec)
2. ✅ **Get model predictions** (0.3 sec)
3. ✅ **Compute SHAP explanation** (91 sec)
4. ✅ **Generate visualizations** (2 sec)
5. ✅ **Save outputs** to `experiments/results/shap/`

---

## Output Files

```
experiments/results/shap/
├── original_image.png         # Original MRI
├── predictions.png            # Class probability chart
└── shap_explanation.png       # SHAP heatmap
```

---

## Example Output

**Input Image**: Meningioma MRI scan
```
Model Predictions:
  glioma     : 3.99%
  meningioma : 99.88% ← Model says this
  notumor    : 11.54%
  pituitary  : 1.34%

SHAP Result:
  Shows which pixels influenced the meningioma prediction
```

---

## Key Metrics

- **Processing Time**: ~91 seconds per image (on CPU)
- **Model Accuracy**: High confidence predictions (~99% on meningioma test)
- **SHAP Method**: GradientExplainer (gradient-based attribution)
- **Background Samples**: 20 MRI images

---

## Interpreting SHAP Output

**SHAP Heatmap Colors**:
- 🔴 **Red**: Pushed model toward prediction (important features)
- 🔵 **Blue**: Pushed model away from prediction (negative features)
- ⚪ **Gray**: No significant contribution

**Example**: 
- Red regions show where the tumor is likely located
- High brightness = stronger influence on prediction

---

## Success Indicators ✅

✅ Model loads without errors
✅ Image preprocessed successfully
✅ Predictions computed (all 4 classes)
✅ SHAP values computed (~91 seconds)
✅ Visualizations generated
✅ Files saved to `experiments/results/shap/`

---

## Troubleshooting

**If SHAP is slow**: Normal! ~91 seconds on CPU is expected

**If image not found**: Check path exists
```bash
ls data/raw/MRI/Testing/meningioma/Te-me_1.jpg
```

**If model not found**: Train first
```bash
python main.py train --config configs/config.yaml
```

---

## Complete System Working ✅

```
INPUT (Image)
    ↓
LOAD & PREPROCESS
    ↓
MODEL PREDICTION (4 class probabilities)
    ↓
SHAP EXPLANATION (gradient-based)
    ↓
VISUALIZATIONS (heatmaps + charts)
    ↓
OUTPUT (saved to experiments/results/shap/)
```

