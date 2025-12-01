# VASA-1-hack Setup and Training Log

## Date: December 1, 2025

## What Was Done

### 1. Fixed Dependencies
- **transformers downgrade**: 4.57.3 → 4.36.0 (to avoid PyTorch 2.6+ requirement from CVE-2025-32434)
- **phonemizer + espeak-ng**: Installed for phoneme processing
- **timm**: Installed for Synchformer's MotionFormer visual feature extraction

### 2. Fixed Configuration Issues
- **Video folder path**: Changed `video_folder: "s1"` to `video_folder: "junk"` in `overfit_config.yaml`
- **Cache directory**: Changed `cache_dir: "cache_per_video"` to `cache_dir: "cache_single_bucket"`

### 3. Preprocessing Completed
- Ran `preprocess_single_bucket.py` on 3 videos in `junk/` folder:
  - VID_1.mp4 (38 windows)
  - VID_2.mp4 (32 windows)
  - 15.mp4 (38 windows)
- Total: **108 windows** preprocessed
- Cache size: **5.0 GB** in `cache_single_bucket/`
  - `all_windows_cache.h5` (3.5GB) - main window data
  - `expression_embeddings.h5` (1.4MB) - 5400 embeddings, 128D
  - `frames/` and `emo_frames/` directories with PNG frames

### 4. Training Started
- Training script: `python train_overfit.py`
- Configuration:
  - Batch size: 8
  - Learning rate: 0.0005
  - Epochs: 4000
  - Window size: 50 frames
  - Stride: 25 frames
  - Mixed precision (fp16) enabled
  - 27 batches per epoch

## Training Time Estimates

| Epochs | Time Estimate | Use Case |
|--------|---------------|----------|
| 100 | ~7.5 hours | Quick overfit test |
| 500 | ~37.5 hours | Moderate overfit validation |
| 4000 | ~300 hours (12.5 days) | Full training |

Each epoch takes ~4.5 minutes (27 batches × ~10 sec/window).

## Disk Space Requirements

### Current Setup (3 videos, overfitting test)
- Video data: ~500MB
- Preprocessed cache: ~5GB
- Checkpoints: ~100MB each (saved periodically)
- Wandb logs: ~10-50MB
- **Minimum**: 20GB free

### Full Training Recommendations

| Dataset Size | Videos | Disk Needed | Notes |
|--------------|--------|-------------|-------|
| Small | 10-50 | 50-100GB | Quick experiments |
| Medium | 100-500 | 200-500GB | Decent generalization |
| Large | 1000+ | 1-2TB | Production quality |

**Formula**: ~50MB per video (cache) + ~100MB per checkpoint × checkpoints saved

### Recommended Server Specs for Full Training

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| GPU | RTX 3090 (24GB) | A100 (40/80GB) |
| VRAM | 24GB | 40-80GB |
| Disk | 500GB SSD | 1-2TB NVMe |
| RAM | 32GB | 64GB |
| CPU | 8 cores | 16+ cores |

## Commands to Run

```bash
# Activate environment
conda activate vasa

# Preprocess videos (run once)
python preprocess_single_bucket.py \
    --video_folder junk/ \
    --cache_dir cache_single_bucket \
    --max_videos 3

# Start training
python train_overfit.py

# Monitor training (in another terminal)
tail -f wandb/latest-run/files/output.log
```

## Current Status

- **Branch**: `nemo`
- **Training**: Running (Epoch 0)
- **GPU utilization**: ~97%
- **VRAM usage**: 19.8GB/23GB
- **Initial loss**: 18-24 (expected to decrease)

## Known Issues

1. **Synchformer checkpoint not found**: Using random initialization (warning only)
2. **Loss warnings in early training**: Normal - losses start high and decrease
3. **Submodules modified**: `face-detection` and `nemo` have local changes

## Files Modified

- `overfit_config.yaml` - Fixed paths for video_folder and cache_dir
- `train_overfit.py` - No changes needed (was already correct)
