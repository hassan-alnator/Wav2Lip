# Fast Blending Enhancement Branch

This branch contains performance-optimized blending methods for Wav2Lip that eliminate the 96x96 box artifact without requiring model retraining or heavy dependencies.

## What's New

- **4 Fast Blending Methods**: From 1ms to 15ms per frame overhead
- **No External Dependencies**: Works with standard Wav2Lip requirements
- **No Model Retraining**: Uses existing Wav2Lip checkpoints
- **Real-time Capable**: All methods fast enough for live applications

## Quick Start

```bash
# Switch to this branch
git checkout fast-blending-enhancement

# Run with fast elliptical blending (default)
python inference_fast.py \
  --checkpoint_path checkpoints/wav2lip_gan.pth \
  --face video.mp4 \
  --audio audio.wav \
  --outfile output.mp4
```

## Available Methods

1. **ellipse** - 1-2ms overhead, good quality
2. **multiscale** - 3-5ms overhead, better quality  
3. **edge_aware** - 5-10ms overhead, very good quality
4. **guided** - 10-15ms overhead, best quality

## Documentation

See [FAST_BLENDING_GUIDE.md](FAST_BLENDING_GUIDE.md) for complete documentation, examples, and optimization tips.

## Comparison with Master Branch

| Feature | Master Branch | This Branch |
|---------|--------------|-------------|
| Blending Methods | Feather/Poisson | 4 optimized methods |
| Speed | Moderate | Very Fast (1-15ms) |
| Mask Shape | Rectangular | Elliptical |
| Real-time Ready | Partially | Fully |

## Files Added

- `inference_fast.py` - Enhanced inference script with fast blending
- `FAST_BLENDING_GUIDE.md` - Complete documentation
- `README_BRANCH.md` - This file