# Quick Reference Card

## 🚀 Quick Start (5 minutes)

```bash
# 1. Activate environment
source venv/bin/activate

# 2. Verify setup
python test_setup.py

# 3. Train model
python main.py train

# 4. Generate explanations
python main.py explain \
    --model_path models/checkpoints/best_model.pth \
    --image_path data/raw/NIH_ChestXray/images_001/images/00000001_000.png
```

## 📋 Common Commands

| Task | Command |
|------|---------|
| **Setup verification** | `python test_setup.py` |
| **Train model** | `python main.py train --config configs/config.yaml` |
| **Generate SHAP** | `python main.py explain --model_path ... --image_path ...` |
| **Example analysis** | `python example_analysis.py` |
| **View logs** | `tail -f logs/main.log` |
| **Check GPU** | `python -c "import torch; print(torch.cuda.is_available())"` |

## 🎯 Main Entry Points

### Training
```python
python main.py train --config configs/config.yaml
```
- Loads dataset
- Trains ResNet50 for 10 epochs
- Saves model to `models/checkpoints/best_model.pth`
- Saves history to `experiments/results/training_history.json`

### Explanation
```python
python main.py explain \
    --model_path models/checkpoints/best_model.pth \
    --image_path <path_to_image> \
    --output_dir experiments/results/shap
```
- Loads trained model
- Computes SHAP values
- Generates visualizations
- Saves to `experiments/results/shap/`

## 🔧 Configuration (config.yaml)

```yaml
# Quick adjustments
training:
  batch_size: 32          # Reduce for OOM errors
  num_epochs: 10          # Increase for better accuracy
  learning_rate: 0.001    # Adjust convergence speed

dataset:
  image_size: 224         # Input image size
  num_classes: 14         # Disease classes

shap:
  background_size: 50     # Background samples
```

## 📦 Project Structure

```
models/
├── cnn_model.py         # ResNet50 architecture
├── train.py             # Training pipeline
└── checkpoints/         # Model weights

xai/
├── base_explainer.py    # Explainer interface
└── shap_explainer.py    # SHAP implementation

utils/
├── dataset_loader.py    # Dataset handling
├── preprocessing.py     # Image preprocessing
└── visualization.py     # Visualization tools

evaluation/
└── metrics.py           # Evaluation metrics

configs/
└── config.yaml          # Configuration
```

## 🔍 Key Classes

### Model
```python
from models import get_model, load_model

# Create model
model = get_model(num_classes=14, device='cuda')

# Load trained model
model = load_model('models/checkpoints/best_model.pth', device='cuda')

# Forward pass
logits = model(images)  # Shape: (batch, 14)
probs = torch.sigmoid(logits)  # Probabilities
```

### Dataset
```python
from utils.dataset_loader import ChestXrayDataLoader

loader = ChestXrayDataLoader('data/raw/NIH_ChestXray', 'csv_path')

# Train/Val split (90/10)
train_loader, val_loader = loader.get_train_val_loaders(...)

# Test split
test_loader = loader.get_test_loader(...)

# Background for SHAP
bg_loader = loader.get_background_loader(...)
```

### SHAP Explainer
```python
from xai.shap_explainer import SHAPExplainer

explainer = SHAPExplainer(
    background_loader=bg_loader,
    num_samples=50,
    device='cuda'
)

explanation = explainer.explain(image, model, target_class=None)
# Returns: {
#   'attributions': (H, W) heatmap,
#   'predictions': (14,) probabilities,
#   'shap_values': raw SHAP values,
# }
```

### Metrics
```python
from evaluation.metrics import MultiLabelMetrics

# Individual metrics
auroc = MultiLabelMetrics.compute_auroc(predictions, labels)
auprc = MultiLabelMetrics.compute_auprc(predictions, labels)
f1 = MultiLabelMetrics.compute_f1_score(predictions, labels)

# Per-class metrics
metrics = MultiLabelMetrics.compute_per_class_metrics(
    predictions, labels, disease_classes
)
```

## 📊 Disease Classes (14 total)

1. Atelectasis
2. Cardiomegaly
3. Effusion
4. Infiltration
5. Mass
6. Nodule
7. Pneumonia
8. Pneumotharax
9. Consolidation
10. Edema
11. Emphysema
12. Fibrosis
13. Pleural_Thickening
14. Hernia

## 🐛 Troubleshooting

| Problem | Solution |
|---------|----------|
| CUDA out of memory | Reduce `batch_size` in config |
| Slow data loading | Increase `num_workers` in config |
| Model not found | Train first: `python main.py train` |
| Dataset not found | Download from NIH and place in `data/raw/NIH_ChestXray/` |
| Import errors | Install dependencies: `pip install -r requirements.txt` |

## 📈 Expected Performance

| Hardware | Batch Size | Epochs | Time |
|----------|-----------|--------|------|
| M1 Pro | 8 | 10 | ~4 hours |
| NVIDIA A100 | 64 | 10 | ~30 min |
| CPU | 8 | 10 | ~8 hours |

## 💾 File Locations

| File | Location |
|------|----------|
| Dataset | `data/raw/NIH_ChestXray/` |
| Model weights | `models/checkpoints/best_model.pth` |
| Results | `experiments/results/` |
| SHAP outputs | `experiments/results/shap/` |
| Logs | `logs/main.log` |
| Config | `configs/config.yaml` |

## 🔑 Key Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| torch | 2.1.2 | Deep learning framework |
| torchvision | 0.16.2 | Computer vision utilities |
| shap | 0.43.1 | Explainability |
| numpy | 1.24.3 | Numerical computing |
| pandas | 2.0.3 | Data manipulation |
| scikit-learn | 1.3.0 | Machine learning utilities |
| matplotlib | 3.7.2 | Visualization |

## 📝 Git Commands

```bash
# Check what will be tracked
git status

# Commit code (not data)
git add main.py models/ xai/ utils/ evaluation/
git commit -m "Add XAI project"

# Push to GitHub
git push origin main

# Verify data is not tracked
git check-ignore -v data/raw/
```

## 🎓 Example Workflow

```python
import torch
from models import get_model, load_model
from utils.dataset_loader import ChestXrayDataLoader
from xai.shap_explainer import SHAPExplainer
from evaluation.metrics import MultiLabelMetrics

# 1. Load model
model = load_model('models/checkpoints/best_model.pth', device='cuda')

# 2. Load dataset
loader = ChestXrayDataLoader('data/raw/NIH_ChestXray', 'csv_path')
bg_loader = loader.get_background_loader(...)

# 3. Explain
explainer = SHAPExplainer(background_loader=bg_loader, device='cuda')
explanation = explainer.explain(image, model)

# 4. Evaluate
auroc = MultiLabelMetrics.compute_auroc(predictions, labels)
```

## 📚 Documentation

- **README.md** - Full documentation
- **SETUP_GUIDE.md** - Setup instructions
- **PROJECT_STRUCTURE.py** - Detailed structure
- **configs/config.yaml** - Configuration options

---

**Need help?** Check `logs/main.log` or run `python test_setup.py`
