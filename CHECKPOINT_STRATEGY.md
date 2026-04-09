## ✅ Model Checkpoint Saving Strategy

**Short Answer**: Yes, if you stop training mid-epoch, the **best model from the PREVIOUS completed epoch** will still exist and be usable.

---

## How Checkpointing Works

### 📋 Checkpoint Timing

The model saves checkpoints at **specific times**, NOT continuously:

1. **After Each Epoch Completes** (Best Model)
   - After validation is done, if the model is better than before
   - File: `models/checkpoints/best_model.pth`
   - Triggered when: `validation_loss < best_validation_loss_so_far`

2. **Every 5 Epochs** (Periodic Checkpoint)
   - After epochs 5, 10, 15, 20, etc.
   - Files: `models/checkpoints/checkpoint_epoch_5.pth`, `checkpoint_epoch_10.pth`, etc.
   - Useful for safety (if you want to restart from a specific epoch)

### ⚠️ What Happens If You Stop Mid-Epoch

**If you stop during training (e.g., epoch 3 is only 50% done):**

✅ **These WILL exist**:
- `best_model.pth` - Best model from epochs 1-2 (fully completed)
- Any checkpoint files from epochs 1-2 (if epoch 2 was the 5th, 10th, etc.)

❌ **These WON'T be saved**:
- Incomplete epoch 3 (no checkpoint for partial epoch)
- No progress from epoch 3 is saved

---

## Checkpoint Files Explained

### `best_model.pth`
- **When saved**: After each epoch IF validation loss improved
- **Contains**: 
  - Model weights
  - Optimizer state
  - Epoch number
  - Validation metrics (loss, AUROC)
- **Usage**: Resume training or final inference
- **Example**: If epochs 1-3 completed but epoch 4 stopped mid-way, `best_model.pth` contains the best model from epochs 1-3

### `checkpoint_epoch_5.pth`, `checkpoint_epoch_10.pth`, etc.
- **When saved**: After epochs 5, 10, 15, 20, etc. complete
- **Contains**: Same as best_model.pth
- **Usage**: Full training resumption from that exact epoch
- **Example**: After epoch 10 completes, `checkpoint_epoch_10.pth` is saved with all 10 epochs of training

---

## Practical Examples

### Example 1: Stop During Epoch 3

```
Epoch 1: ✅ COMPLETE → best_model.pth saved (if improved)
Epoch 2: ✅ COMPLETE → best_model.pth updated (if improved)
Epoch 3: ⏹️  STOPPED at 50% → No checkpoint for epoch 3
```

**Result**: 
- ✅ `best_model.pth` exists with best model from epochs 1-2
- ❌ Epoch 3 progress is lost
- ❌ `checkpoint_epoch_5.pth` doesn't exist yet

### Example 2: Stop During Epoch 7

```
Epoch 1: ✅ COMPLETE
Epoch 2: ✅ COMPLETE
Epoch 3: ✅ COMPLETE
Epoch 4: ✅ COMPLETE
Epoch 5: ✅ COMPLETE → checkpoint_epoch_5.pth saved
Epoch 6: ✅ COMPLETE
Epoch 7: ⏹️  STOPPED at 30% → No checkpoint for epoch 7
```

**Result**:
- ✅ `best_model.pth` exists with best model from epochs 1-6
- ✅ `checkpoint_epoch_5.pth` exists (can resume from epoch 6)
- ❌ Epoch 7 progress is lost

---

## Files Saved During Training

```
models/checkpoints/
├── best_model.pth              (Best model so far - always updated)
├── checkpoint_epoch_5.pth      (Checkpoint after epoch 5)
├── checkpoint_epoch_10.pth     (Checkpoint after epoch 10)
└── checkpoint_epoch_15.pth     (Checkpoint after epoch 15, if training goes that far)
```

---

## Recovery Strategy

### If You Stop Training Mid-Epoch

1. **The best model up to that point is saved**
   ```bash
   # Use the best model for inference
   python main.py explain \
     --model_path models/checkpoints/best_model.pth \
     --image_path data/raw/MRI/Testing/glioma/Te-gl_1.jpg
   ```

2. **You lose only the current incomplete epoch**
   - All completed epochs are preserved
   - Start from scratch or from a checkpoint to continue

### If You Want to Resume Training from Epoch 5

Currently, resuming from a specific checkpoint requires code modification. The current implementation always trains from scratch but loads the best model saved so far.

**Current behavior**: If you run training again, it starts from epoch 1 but won't overwrite the `best_model.pth` unless it finds a better model.

---

## Key Takeaways

| Scenario | Model Saved? | Loss? | How to Use |
|----------|------------|------|-----------|
| Stop at epoch 1 (50%) | ✅ (Epoch 0) | ❌ Epoch 1 training | `best_model.pth` is usable |
| Stop at epoch 3 (end) | ✅ (Epoch 3) | ✅ None | `best_model.pth` has 3 epochs |
| Stop at epoch 5 (end) | ✅ (Epoch 5) | ✅ None | `best_model.pth` + `checkpoint_epoch_5.pth` |
| Stop at epoch 7 (50%) | ✅ (Epoch 6) | ❌ Epoch 7 training | `best_model.pth` has 6 epochs |
| Stop at epoch 10 (end) | ✅ (Epoch 10) | ✅ None | Full training + `checkpoint_epoch_10.pth` |

---

## Recommendation

**To minimize loss if stopping mid-training:**
- Stop at the end of an epoch (after validation completes) rather than mid-epoch
- The complete epoch will be saved in `best_model.pth`
- You can then use that model or resume training fresh

---

## Technical Details

**Code Location**: `models/train.py` (lines 253-290)

```python
# Saves best model after each epoch validation
if val_loss < self.best_val_loss:
    self.best_val_loss = val_loss
    save_model(self.model, 'best_model.pth', ...)

# Saves periodic checkpoint every 5 epochs
if (epoch + 1) % 5 == 0:
    save_model(self.model, f'checkpoint_epoch_{epoch + 1}.pth', ...)
```

---

**In summary**: Your model is safe! Checkpoints are saved after each complete epoch, so stopping mid-epoch only loses that incomplete epoch's work. 💾

