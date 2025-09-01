# Wav2Lip Enhanced - Project Context for Claude

## Project Overview
This is an enhanced fork of Wav2Lip that solves production issues with the original implementation. The user runs an app/service that uses this for avatar video generation with lip-sync.

## Key Requirements
- **Performance Target**: 10-15 seconds max processing time (critical!)
- **Quality Issues**: Original Wav2Lip has visible 96x96 box artifact and mouth distortion
- **Avatar Specifics**: User's avatar has a mustache which causes additional challenges
- **Integration**: Used via command line from a larger application

## Current Solution Architecture

### Branch: fast-blending-enhancement
All enhancements are on this branch, NOT master.

### Main Script: inference_fast.py
This is the production script with all enhancements:
- Multiple blend modes (ellipse, multiscale, edge_aware, guided, hd)
- Sharpening capabilities
- Temporal smoothing (experimental - causes issues)
- HD enhancement mode

### Key Files Modified
1. **inference_fast.py** - Main enhanced inference script
2. **inference_hd_fast.py** - HD enhancement functions
3. **FAST_BLENDING_GUIDE.md** - Complete documentation
4. **README.md** - Updated to highlight improvements

## Known Issues & Solutions

### 1. Mustache Causing Mouth Distortion
**Problem**: Wav2Lip trained on clean faces, mustache interferes
**Solution**: Use smaller mouth_region_size (0.4 or less) to focus on lips only
```bash
--mouth_region_size 0.4
```

### 2. Temporal Smoothing Issues
**Problem**: Causes slow motion effect and makes quality worse
**Status**: Marked as EXPERIMENTAL, don't use in production
**Avoid**: Don't use --temporal_smooth flag

### 3. HD Mode Implementation
**Initial Problem**: HD mode wasn't actually enhancing (was using already-resized patch)
**Fix Applied**: Now uses original 96x96 patch with 7-step enhancement process
**Current Status**: Working, adds ~15-20ms but provides visible improvement

## Production Commands

### FASTEST CONFIGURATION (30-50% speedup with FP16 + Silence Gating)
```bash
python inference_fast.py \
  --checkpoint_path checkpoints/wav2lip_gan.pth \
  --face video.mp4 \
  --audio audio.wav \
  --fp16 \                    # 30-50% speed boost, no quality loss
  --silence_gating \           # Skip 20-30% of frames (silent ones)
  --silence_threshold 0.01 \   # Adjust if needed (0.001-0.1)
  --blend_method ellipse \     # Fast blend (1-2ms)
  --mouth_region_size 0.4 \
  --sharpen_mouth \
  --sharpen_amount 0.6 \
  --outfile output.mp4
```

### Optimized for Mustache + SPEED
```bash
python inference_fast.py \
  --checkpoint_path checkpoints/wav2lip.pth \  # Non-GAN sometimes better
  --face video.mp4 \
  --audio audio.wav \
  --fp16 \                    # CRUCIAL: 30-50% faster
  --silence_gating \           # CRUCIAL: Skip silent frames
  --blend_method ellipse \     # Fast, not HD (saves 15ms)
  --mouth_region_size 0.4 \    # Smaller for mustache
  --blur_intensity 41 \
  --sharpen_mouth \
  --sharpen_amount 0.5 \
  --outfile output.mp4
```

### Original HD Mode (slower but highest quality)
```bash
python inference_fast.py \
  --checkpoint_path checkpoints/wav2lip_gan.pth \
  --face video.mp4 \
  --audio audio.wav \
  --blend_method hd \
  --mouth_region_size 0.6 \
  --blur_intensity 61 \
  --sharpen_mouth \
  --sharpen_amount 0.7 \
  --outfile output.mp4
```

## Technical Details

### Blend Methods Performance
- **ellipse**: 1-2ms overhead, good quality, reliable
- **hd**: 15-20ms overhead, best quality, worth it for production
- **multiscale**: 3-5ms, good but not as good as HD
- **guided/edge_aware**: 5-15ms, situational

### HD Enhancement Process (7 steps)
1. 2x upscale to 192x192 (CUBIC)
2. Edge detection and enhancement
3. Bilateral filtering
4. Resize to target (LANCZOS4)
5. Multi-pass sharpening
6. Adaptive histogram equalization (CLAHE)
7. Color correction

### What Makes This Fork Better
1. **Eliminates 96x96 box artifact** - Advanced blending with feathered masks
2. **Fixes mouth distortion** - Better mask shapes (elliptical vs rectangular)
3. **Sharper output** - Sharpening filters and HD mode
4. **Production ready** - Fast enough for real-time use

## Important Notes for Future Claude Sessions

### DO NOT
- Don't suggest temporal smoothing (it's broken and makes things worse)
- Don't suggest Real-ESRGAN or heavy super-resolution (too slow)
- Don't suggest Speech2Lip or other alternatives (too slow/complex)
- Don't break backward compatibility (app integration depends on current API)

### ALWAYS
- Keep performance under 15 seconds
- Test that HD mode uses the ORIGINAL 96x96 patch (not pre-resized)
- Maintain the current command structure (no API changes)
- Consider the mustache issue when suggesting parameters

### Performance Constraints
- Target: 10-15 seconds total
- GPU: Available and used
- Batch size: Can be adjusted but default works
- No external model downloads acceptable

## User's Workflow
1. App calls inference_fast.py via command line
2. Expects video output in ~15 seconds
3. No changes to app integration desired
4. Quality improvements welcome if performance maintained

## Recent Fixes Applied
1. Fixed size mismatch errors in temporal smoothing
2. Fixed HD mode not actually enhancing (was using wrong patch)
3. Removed redundant resize operations for speed
4. Optimized blend methods for performance
5. **Added FP16 optimization** - 30-50% speed boost with no quality loss
6. **Added silence gating** - Skips 20-30% of frames during silence

## Testing Approach
When testing changes:
1. Compare regular vs HD mode visually
2. Check processing time stays under 15 seconds
3. Verify mustache doesn't cause artifacts
4. Ensure backward compatibility maintained

## Contact Context
User is direct, wants working solutions quickly, doesn't need long explanations. Focus on:
- What changed
- How to use it
- Whether app changes needed (usually NO)
- Actual performance impact

Remember: This is production code for a real service. Stability and performance are critical.