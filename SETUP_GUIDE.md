# Setup Guide

## System Requirements

- **Python**: 3.10 or higher
- **OS**: macOS, Linux, or Windows
- **RAM**: 16GB minimum (32GB recommended)
- **GPU**: NVIDIA GPU with CUDA support (optional but recommended)
- **Storage**: 500GB+ for dataset and model checkpoints

## Step-by-Step Setup

### 1. Verify Python Installation

```bash
python --version  # Should be 3.10+
python -m venv --help  # Verify venv is available
```

### 2. Create Virtual Environment

```bash
cd /Users/nihaldastagiri/Desktop/XAI
python -m venv venv
source venv/bin/activate  # On macOS/Linux
# Or on Windows: venv\Scripts\activate
```

### 3. Upgrade pip

```bash
pip install --upgrade pip setuptools wheel
```

### 4. Install Dependencies

```bash
pip install -r requirements.txt
```

**Installation time**: ~5-10 minutes

**What gets installed**:
- PyTorch 2.1.2 with CUDA support
- torchvision 0.16.2
- NumPy, Pandas, Scikit-learn
- SHAP 0.43.1
- YAML, Pillow, tqdm

### 5. Verify Installation

```bash
python test_setup.py
```

**Expected output**:
```
✓ Directory Structure
✓ Imports
✓ Dataset
✓ Configuration
✓ Model
✓ ALL TESTS PASSED - READY TO TRAIN!
```

## GPU Setup (Optional but Recommended)

### macOS (Silicon - M1/M2/M3)

PyTorch automatically uses Metal Performance Shaders:
```bash
python -c "import torch; print(torch.backends.mps.is_available())"
```

### NVIDIA GPU (Linux/Windows)

Check CUDA availability:
```bash
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"
```

If CUDA is not detected:
```bash
# Install CUDA-enabled PyTorch
pip uninstall torch
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

## Dataset Verification

After downloading the dataset, verify structure:

```bash
cd data/raw/NIH_ChestXray

# Check required files exist
ls -la Data_Entry_2017.csv
ls -la train_val_list.txt
ls -la test_list.txt

# Check image folders
ls -d images_001 images_002 ... images_012

# Count total images
find . -name "*.png" | wc -l  # Should be ~112,000+
```

## Directory Structure Creation

The project automatically creates directories as needed:
- `logs/` - Training logs
- `models/checkpoints/` - Model weights
- `experiments/results/` - Output files
- `data/processed/` - Processed data
- `data/annotations/` - Annotations

## Git Configuration

### Initialize Git (if not already done)

```bash
git init
git add .gitignore
git add *.py configs/ models/ xai/ utils/ evaluation/
git add requirements.txt README.md
git commit -m "Initial XAI project setup"
```

### Verify .gitignore

```bash
git check-ignore -v data/raw/
# Should output: data/ .gitignore
```

## Configuration

### Default Configuration

Edit `configs/config.yaml` to customize:

**For Quick Testing** (single GPU):
```yaml
training:
  batch_size: 16
  num_epochs: 2
  num_workers: 2
```

**For Full Training** (multiple GPUs if available):
```yaml
training:
  batch_size: 64
  num_epochs: 10
  num_workers: 8
```

**For CPU Only**:
```yaml
training:
  batch_size: 8
  num_epochs: 5
  num_workers: 0
```

## Troubleshooting

### Issue: `ModuleNotFoundError: No module named 'torch'`

**Solution**:
```bash
# Ensure virtual environment is activated
which python  # Should show path inside venv/

# Reinstall PyTorch
pip install torch torchvision
```

### Issue: `CUDA out of memory`

**Solution**:
```yaml
# Reduce batch size in config.yaml
training:
  batch_size: 8  # Reduce from 32
```

Or use CPU:
```bash
# Set environment variable
CUDA_VISIBLE_DEVICES='' python main.py train --config configs/config.yaml
```

### Issue: `FileNotFoundError: data/raw/NIH_ChestXray`

**Solution**: Verify dataset is downloaded:
```bash
ls -la /Users/nihaldastagiri/Desktop/XAI/data/raw/NIH_ChestXray/
```

Download from: https://nihcc.app.box.com/v/ChestXray-NIHCC

### Issue: Slow data loading

**Solution**: Increase workers:
```yaml
training:
  num_workers: 8  # Increase from 4
```

### Issue: ImportError with SHAP

**Solution**:
```bash
pip install --upgrade shap
```

## Testing the Setup

### Quick Test (5 minutes)

```bash
# Test all components
python test_setup.py

# Test single model forward pass
python -c "
import torch
from models import get_model
model = get_model(device='cpu')
x = torch.randn(1, 3, 224, 224)
y = model(x)
print('Model output shape:', y.shape)
"
```

### Load Sample Data

```bash
python -c "
from utils.dataset_loader import ChestXrayDataLoader
import os

dataset_dir = 'data/raw/NIH_ChestXray'
csv_path = os.path.join(dataset_dir, 'Data_Entry_2017.csv')

loader = ChestXrayDataLoader(dataset_dir, csv_path)
print('✓ Dataset loader initialized')
print('Number of images in CSV:', len(loader.labels_df))
"
```

## Environment Variables

### For GPU Usage

```bash
# Use specific GPU
export CUDA_VISIBLE_DEVICES=0

# For memory optimization
export CUDA_LAUNCH_BLOCKING=1

# For debugging
export TORCH_USE_CUDA_DSA=1
```

### For CPU Only

```bash
export CUDA_VISIBLE_DEVICES=''
```

## Logging

View training logs in real-time:

```bash
# Terminal 1: Start training
python main.py train

# Terminal 2: Monitor logs
tail -f logs/main.log
```

## Next Steps After Setup

1. **Verify setup**: `python test_setup.py`
2. **Train model**: `python main.py train --config configs/config.yaml`
3. **Generate explanations**: `python main.py explain --model_path models/checkpoints/best_model.pth --image_path <image_path>`
4. **Run example**: `python example_analysis.py`

## Performance Benchmarks

Expected training times on different hardware:

| Hardware | Batch Size | Epochs | Time |
|----------|-----------|--------|------|
| M1 Pro (8 cores) | 8 | 10 | ~4 hours |
| M1 Max (10 cores) | 16 | 10 | ~2.5 hours |
| NVIDIA A100 | 64 | 10 | ~30 min |
| NVIDIA RTX 3090 | 64 | 10 | ~1 hour |
| CPU (Intel i7) | 8 | 10 | ~8 hours |

## Memory Requirements

| Component | Memory |
|-----------|--------|
| Model (ResNet50) | ~500 MB |
| Batch of 32 images | ~1.5 GB |
| Validation set | ~10 GB |
| SHAP processing (50 bg) | ~5 GB |
| **Total (training)** | **~20 GB** |

## Useful Commands

```bash
# Activate environment
source venv/bin/activate

# Deactivate environment
deactivate

# Install additional packages
pip install package_name

# Freeze requirements (save current environment)
pip freeze > requirements_lock.txt

# Check GPU status (NVIDIA)
nvidia-smi

# Monitor CPU/Memory
top -l 1 | head -20
```

## System Maintenance

```bash
# Clean pip cache
pip cache purge

# Remove unused packages
pip autoremove

# Update all packages
pip list --outdated
pip install --upgrade <package>
```

## Getting Help

1. Check `logs/main.log` for error messages
2. Run `python test_setup.py` to diagnose setup issues
3. Verify dataset files exist
4. Check `README.md` for API reference
5. Review configuration in `configs/config.yaml`

---

**Setup complete!** You're ready to train and explain models.
