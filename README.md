# XAI Techniques Analysis for Chest X-ray Classification

A comprehensive, production-quality Python project for evaluating explainable AI (XAI) techniques on medical imaging using the NIH ChestX-ray14 dataset.

## 🎯 Project Overview

This project builds a complete pipeline for:
1. **Training** a CNN model (ResNet50) on the NIH ChestX-ray14 dataset
2. **Multi-label classification** of 14 chest diseases
3. **Explainability** using SHAP (SHapley Additive exPlanations)
4. **Evaluation** with medical imaging metrics

The architecture is modular and extensible, designed to support additional XAI techniques (LIME, Grad-CAM) in the future.

## 📦 Dataset

The project uses the **NIH ChestX-ray14 dataset** already downloaded and located at:
```
data/raw/NIH_ChestXray/
```

### Dataset Structure
- **Images**: Distributed across `images_001/` to `images_012/` directories
- **Labels**: `Data_Entry_2017.csv` (14 disease classes)
- **Splits**: `train_val_list.txt`, `test_list.txt`
- **Annotations**: `BBox_List_2017.csv` (bounding boxes for localization)

### Disease Classes (14 total)
1. Atelectasis
2. Cardiomegaly
3. Effusion
4. Infiltration
5. Mass
6. Nodule
7. Pneumonia
8. Pneumothorax
9. Consolidation
10. Edema
11. Emphysema
12. Fibrosis
13. Pleural_Thickening
14. Hernia

## 📁 Project Structure

```
XAI/
├── data/                                  # Dataset (not tracked by git)
│   ├── raw/NIH_ChestXray/
│   ├── processed/
│   └── annotations/
│
├── models/
│   ├── __init__.py
│   ├── cnn_model.py                      # ResNet50 architecture
│   ├── train.py                          # Training pipeline
│   └── checkpoints/                      # Model weights
│
├── xai/
│   ├── __init__.py
│   ├── base_explainer.py                 # Base explainer interface
│   └── shap_explainer.py                 # SHAP implementation
│
├── utils/
│   ├── __init__.py
│   ├── dataset_loader.py                 # Dataset handling
│   ├── preprocessing.py                  # Image preprocessing
│   └── visualization.py                  # Visualization utilities
│
├── evaluation/
│   ├── __init__.py
│   └── metrics.py                        # Performance metrics
│
├── experiments/
│   └── results/                          # Outputs and visualizations
│
├── configs/
│   └── config.yaml                       # Configuration file
│
├── logs/                                 # Training logs
├── main.py                               # Entry point
├── requirements.txt                      # Dependencies
├── .gitignore                            # Git ignore rules
├── README.md                             # This file
└── LICENSE
```

## ⚙️ Installation

### 1. Clone and Navigate
```bash
cd /Users/nihaldastagiri/Desktop/XAI
```

### 2. Create Virtual Environment (Recommended)
```bash
python3.10 -m venv venv
source venv/bin/activate  # On macOS/Linux
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Verify Installation
```bash
python -c "import torch; print(f'PyTorch: {torch.__version__}'); print(f'CUDA Available: {torch.cuda.is_available()}')"
```

## 🚀 Quick Start

### Training the Model

```bash
python main.py train --config configs/config.yaml
```

**What this does:**
1. Loads the NIH ChestX-ray dataset
2. Splits data into train (90%) and validation (10%)
3. Creates ResNet50 model with 14 outputs
4. Trains for 10 epochs with Adam optimizer
5. Saves best model to `models/checkpoints/best_model.pth`
6. Logs training history to `experiments/results/training_history.json`

**Expected output:**
```
Training loss: 0.1234
Validation loss: 0.1456, AUROC: 0.8765
Best model saved at epoch 5 with loss 0.1234
```

### Generating SHAP Explanations

```bash
python main.py explain \
    --config configs/config.yaml \
    --model_path models/checkpoints/best_model.pth \
    --image_path data/raw/NIH_ChestXray/images_001/images/00000001_000.png
```

**What this does:**
1. Loads trained model
2. Prepares background images (50 samples) for SHAP
3. Computes SHAP values for the input image
4. Generates visualizations:
   - `predictions.png`: Model predictions bar chart
   - `shap_explanation.png`: Original image + heatmap + overlay
   - `original_image.png`: Input image

**Output saved to:** `experiments/results/shap/`

## ⚙️ Configuration

The project is controlled by `configs/config.yaml`:

```yaml
# Dataset
dataset:
  path: "data/raw/NIH_ChestXray"
  image_size: 224
  num_classes: 14

# Training
training:
  batch_size: 32
  num_epochs: 10
  learning_rate: 0.001
  warmup_epochs: 2

# SHAP
shap:
  background_size: 50
  num_samples: 100
```

Modify this file to customize:
- Batch size and learning rate
- Number of training epochs
- SHAP background size
- Number of workers for data loading

## 🧠 Model Architecture

**ResNet50ChestXray**
- **Backbone**: Pretrained ResNet50 (ImageNet weights)
- **Output Layer**: Fully connected layer with 14 units
- **Activation**: Sigmoid (for multi-label classification)
- **Loss Function**: BCEWithLogitsLoss

```python
from models import get_model
model = get_model(num_classes=14, device='cuda')
```

### Model Summary
```
ResNet50ChestXray
├── Backbone: ResNet50 (2048 features)
├── Dropout: 50%
└── FC Layer: 2048 → 14

Total parameters: ~23.7M
Trainable parameters: ~23.7M
```

## 📊 Data Pipeline

### Dataset Loader

Automatically handles:
- ✅ Scanning all 12 image folders
- ✅ Building image-to-label mappings
- ✅ Multi-label encoding (14 disease classes)
- ✅ Train/val/test splits
- ✅ Image preprocessing and augmentation

```python
from utils.dataset_loader import ChestXrayDataLoader

loader = ChestXrayDataLoader(
    dataset_dir='data/raw/NIH_ChestXray',
    csv_path='data/raw/NIH_ChestXray/Data_Entry_2017.csv'
)

train_loader, val_loader = loader.get_train_val_loaders(
    train_val_list_path='data/raw/NIH_ChestXray/train_val_list.txt',
    batch_size=32
)
```

### Preprocessing

**Training:**
- Resize to 224×224
- Horizontal flip (50%)
- Rotation (±15°)
- Color jitter
- Normalization (ImageNet mean/std)

**Inference:**
- Resize to 224×224
- Normalization only

```python
from utils.preprocessing import preprocess_image, denormalize_image

image = preprocess_image('path/to/image.png', image_size=224, augment=False)
denorm = denormalize_image(image)  # For visualization
```

## 🔍 SHAP Explainer

### What is SHAP?

SHAP (SHapley Additive exPlanations) provides:
- **Feature Importance**: Which pixels matter for predictions?
- **Local Explanations**: Why did the model make this specific prediction?
- **Global Consistency**: Unified explanation framework

### How It Works

1. **Background Dataset**: Sample ~50 training images as baseline
2. **Model Masking**: Iteratively mask regions of the input
3. **Shapley Values**: Compute contribution of each pixel
4. **Heatmap**: Visualize as overlay on original image

### Usage

```python
from xai.shap_explainer import SHAPExplainer

explainer = SHAPExplainer(
    background_loader=background_loader,
    num_samples=50,
    device='cuda'
)

explanation = explainer.explain(
    image=image_tensor,
    model=model,
    target_class=None  # Average over all classes
)

# Get results
heatmap = explanation['attributions']  # (H, W)
predictions = explanation['predictions']  # (14,)
```

### Visualization

```python
from utils.visualization import overlay_heatmap

overlay_heatmap(
    image=original_image_array,
    heatmap=shap_heatmap,
    alpha=0.5,
    cmap='jet',
    save_path='results/explanation.png'
)
```

## 📈 Evaluation Metrics

### Implemented Metrics

```python
from evaluation.metrics import MultiLabelMetrics

# AUROC (Area Under ROC Curve)
auroc = MultiLabelMetrics.compute_auroc(predictions, labels)

# AUPRC (Area Under Precision-Recall Curve)
auprc = MultiLabelMetrics.compute_auprc(predictions, labels)

# F1 Score
f1 = MultiLabelMetrics.compute_f1_score(predictions, labels, threshold=0.5)

# Accuracy
accuracy = MultiLabelMetrics.compute_accuracy(predictions, labels)

# Per-class metrics
per_class = MultiLabelMetrics.compute_per_class_metrics(
    predictions, labels, disease_classes
)
```

### Sensitivity Analysis

```python
from evaluation.metrics import sensitivity_analysis

sensitivity = sensitivity_analysis(
    model=model,
    image=image_tensor,
    target_class=0,
    perturbation=0.1,
    num_perturbations=100
)

print(f"Mean sensitivity: {sensitivity['mean_sensitivity']:.4f}")
```

## 🔄 Extending the Project

### Adding LIME Explainer

1. Create `xai/lime_explainer.py`:

```python
from xai.base_explainer import Explainer, ExplainerFactory

class LIMEExplainer(Explainer):
    def __init__(self, **kwargs):
        super().__init__('LIME')
    
    def explain(self, image, model, target_class=None):
        # LIME implementation
        pass
    
    def explain_batch(self, images, model, target_class=None):
        # Batch LIME implementation
        pass

ExplainerFactory.register('lime', LIMEExplainer)
```

2. Use in pipeline:

```python
from xai.base_explainer import ExplainerFactory

explainer = ExplainerFactory.create('lime', num_samples=1000)
explanation = explainer.explain(image, model)
```

### Adding Grad-CAM

Similar process in `xai/gradcam_explainer.py`

## 📝 Logging

Training logs are saved to `logs/main.log`:

```
2024-04-09 10:23:45,123 - __main__ - INFO - Starting training...
2024-04-09 10:24:12,456 - models.train - INFO - Epoch 1/10
2024-04-09 10:24:45,789 - models.train - INFO - Training loss: 0.1234
```

Access logs during training:
```bash
tail -f logs/main.log
```

## 🐛 Troubleshooting

### Issue: CUDA Out of Memory

**Solution**: Reduce batch size in `configs/config.yaml`:
```yaml
training:
  batch_size: 16  # Reduce from 32
```

### Issue: Slow Data Loading

**Solution**: Increase number of workers:
```yaml
training:
  num_workers: 8  # Increase from 4
```

### Issue: Image Not Found

**Solution**: Verify image path:
```bash
find data/raw/NIH_ChestXray -name "*.png" | head -5
```

### Issue: Model Training Stalled

**Solution**: Check if GPU is being used:
```python
print(torch.cuda.is_available())
print(torch.cuda.get_device_name(0))
```

## 📚 API Reference

### Models

```python
from models import get_model, load_model, save_model

# Create new model
model = get_model(num_classes=14, device='cuda')

# Load trained model
model = load_model('models/checkpoints/best_model.pth', device='cuda')

# Save model
save_model(model, 'path/to/model.pth', optimizer=optimizer, epoch=10)
```

### Dataset

```python
from utils.dataset_loader import ChestXrayDataLoader

loader = ChestXrayDataLoader('data/raw/NIH_ChestXray', 'data_entry.csv')
train_loader, val_loader = loader.get_train_val_loaders(...)
test_loader = loader.get_test_loader(...)
background_loader = loader.get_background_loader(...)
```

### Preprocessing

```python
from utils.preprocessing import preprocess_image, denormalize_image

image = preprocess_image('path.png', augment=True)
denorm = denormalize_image(image)
```

### XAI

```python
from xai.base_explainer import ExplainerFactory

explainer = ExplainerFactory.create('shap', background_loader=bg_loader)
explanation = explainer.explain(image, model, target_class=None)
```

## 🎓 References

- **ChestX-ray14**: https://nihcc.app.box.com/v/ChestXray-NIHCC
- **ResNet**: He et al., 2015 - https://arxiv.org/abs/1512.03385
- **SHAP**: Lundberg & Lee, 2017 - https://arxiv.org/abs/1705.07874

## 📄 License

This project is provided for educational and research purposes.

## ✨ Features

- ✅ Production-quality code with proper error handling
- ✅ Modular, extensible architecture
- ✅ Comprehensive logging and monitoring
- ✅ GPU support (CUDA)
- ✅ Multi-label classification (14 diseases)
- ✅ SHAP explanations with visualization
- ✅ Extensive evaluation metrics
- ✅ Configuration-driven pipeline
- ✅ Clean, well-documented code
- ✅ Easy to extend (LIME, Grad-CAM ready)

## 🚀 Next Steps

1. **Train the model**: `python main.py train`
2. **Generate explanations**: `python main.py explain --model_path ... --image_path ...`
3. **Analyze results**: Check `experiments/results/` for outputs
4. **Extend**: Add LIME or Grad-CAM using the modular architecture

## 📧 Support

For issues or questions, refer to:
- Configuration: `configs/config.yaml`
- Logs: `logs/main.log`
- Results: `experiments/results/`

---

**Made with ❤️ for XAI and Medical Imaging**
