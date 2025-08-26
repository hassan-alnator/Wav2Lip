# Wav2Lip Blending Enhancements

## Overview
This fork enhances the original Wav2Lip implementation by fixing the "pixelated face box" issue that occurs when the 96x96 generated patch is upscaled and hard-pasted back onto the original frame.

## Problem Solved
The original Wav2Lip replaces the entire detected face region with a low-resolution generated patch, creating a visible blocky artifact around the face. This enhancement blends only the lower face/mouth area using feathered or Poisson compositing, preserving the original high-resolution quality of the upper face.

## New Features

### Blend Modes
Three blending modes are now available via the `--blend` argument:

1. **`replace`** (original behavior)
   - Hard pastes the generated patch
   - Fast but produces visible box artifacts
   - Use: `--blend replace`

2. **`feather`** (default, recommended)
   - Soft alpha blends only the mouth/lower face region
   - Fast and effective at hiding seams
   - Use: `--blend feather`

3. **`poisson`** (highest quality)
   - Uses OpenCV's seamless cloning
   - Slower but most artifact-resistant
   - Use: `--blend poisson`

### Additional Parameters

- **`--feather`** (default: 31)
  - Controls the Gaussian kernel size for feathering
  - Must be an odd number
  - Larger values = softer edges
  - Example: `--feather 41`

- **`--mouth_from`** (default: 0.50)
  - Fraction from top of face patch where mouth mask starts
  - Range: 0.0 to 1.0
  - 0.50 = start blending from middle of face
  - Example: `--mouth_from 0.55` (start lower)

## Usage Examples

### Basic Usage with Feather Blending (Recommended)
```bash
python inference.py \
  --checkpoint_path checkpoints/wav2lip_gan.pth \
  --face input_video.mp4 \
  --audio speech.wav \
  --blend feather \
  --feather 31 \
  --mouth_from 0.50 \
  --pads 0 12 0 0 \
  --outfile output.mp4
```

### High Quality with Poisson Blending
```bash
python inference.py \
  --checkpoint_path checkpoints/wav2lip_gan.pth \
  --face input_video.mp4 \
  --audio speech.wav \
  --blend poisson \
  --mouth_from 0.50 \
  --pads 0 12 0 0 \
  --outfile output.mp4
```

### Portrait Videos (Head Near Top)
For portrait videos where the head is near the top of the frame:
```bash
python inference.py \
  --checkpoint_path checkpoints/wav2lip_gan.pth \
  --face portrait_720x1080.mp4 \
  --audio speech.wav \
  --blend feather \
  --feather 31 \
  --mouth_from 0.50 \
  --pads 0 15 0 0 \
  --wav2lip_batch_size 32 \
  --face_det_batch_size 32 \
  --outfile portrait_output.mp4
```

### Fixed ROI for Consistent Results
If face detection is inconsistent, lock a fixed region of interest:
```bash
python inference.py \
  --checkpoint_path checkpoints/wav2lip_gan.pth \
  --face input_video.mp4 \
  --audio speech.wav \
  --box TOP BOTTOM LEFT RIGHT \
  --blend feather \
  --feather 31 \
  --outfile output.mp4
```

## Tips for Best Results

1. **Seam Still Visible?**
   - Increase feather size: `--feather 41` or `--feather 51`
   - Adjust mask position: `--mouth_from 0.55` or `--mouth_from 0.60`
   - Try Poisson blending: `--blend poisson`

2. **Face Detection Issues?**
   - Adjust padding: `--pads 0 20 0 0` (increase bottom padding)
   - Use fixed box: `--box` with appropriate coordinates
   - Try `--nosmooth` if face detection jumps around

3. **Performance Optimization**
   - Use `--blend feather` for real-time applications (fast)
   - Use `--blend poisson` for highest quality offline processing
   - Adjust batch sizes based on GPU memory

4. **Post-Processing**
   - Encode at high quality to avoid codec artifacts:
   ```bash
   ffmpeg -y -i output.mp4 -c:v libx264 -crf 18 -preset slow -pix_fmt yuv420p final.mp4
   ```

## Technical Details

### Feather Blending Algorithm
1. Creates a mask focused on the lower face region (mouth/jaw area)
2. Applies Gaussian blur to create soft edges
3. Uses alpha blending: `result = mask * generated + (1-mask) * original`
4. Preserves original face quality in non-mouth regions

### Poisson Blending Algorithm
1. Creates a binary mask for the lower face region
2. Uses OpenCV's `seamlessClone` with `NORMAL_CLONE` mode
3. Centers the clone point slightly below face center for optimal results
4. Provides seamless integration but requires more computation

## Comparison with Original

| Aspect | Original | Enhanced |
|--------|----------|----------|
| Face Region | Entire face replaced | Only mouth/lower face blended |
| Visual Quality | Visible 96x96 box | Seamless integration |
| Upper Face | Low resolution | Original high resolution preserved |
| Processing Speed | Fastest | Fast (feather) / Moderate (poisson) |
| Artifact Resistance | Low | High |

## Backward Compatibility
The enhancement is fully backward compatible. Use `--blend replace` to get the original behavior.

## Credits
Enhancement inspired by community feedback and ChatGPT's suggestions for improving the visual quality of Wav2Lip outputs while maintaining performance.