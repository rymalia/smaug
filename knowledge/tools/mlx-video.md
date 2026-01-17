---
title: mlx-video
author: Blaizzy
stars: 47
language: Python
url: https://github.com/Blaizzy/mlx-video
topics:
  - ltx-2
  - ltx-video
archived_from: Twitter (@Prince_Canuma) - Jan 16, 2026
---

# MLX-Video: AI Video Generation for Apple Silicon

The best package for inference and fine-tuning of Image-to-Video and Audio generation models on your Mac using MLX. Optimized for Apple Silicon with the LTX-2 19B parameter model from Lightricks.

## Supported Models

### LTX-2 (New!)

A 19B parameter video generation model from Lightricks. MLX-Video now supports LTX-2 with a two-stage generation pipeline:

1. **Stage 1**: Generate at half resolution with 8 denoising steps
2. **Upsample**: 2x spatial upsampling via LatentUpsampler
3. **Stage 2**: Refine at full resolution with 3 denoising steps
4. **Decode**: VAE decoder converts latents to RGB video

## Installation

### Option 1: pip (requires git)
```bash
pip install git+https://github.com/Blaizzy/mlx-video.git
```

### Option 2: uv (ultra-fast package manager)
```bash
uv pip install git+https://github.com/Blaizzy/mlx-video.git
```

## Usage

### Text-to-Video Generation

Simple example:
```bash
uv run mlx_video.generate --prompt "Two dogs of the poodle breed wearing sunglasses, close up, cinematic, sunset" -n 100 --width 768
```

With custom settings:
```bash
python -m mlx_video.generate \
    --prompt "Ocean waves crashing on a beach at sunset" \
    --height 768 \
    --width 768 \
    --num-frames 65 \
    --seed 123 \
    --output my_video.mp4
```

### CLI Options

| Option | Default | Description |
|--------|---------|-------------|
| `--prompt`, `-p` | (required) | Text description of the video |
| `--height`, `-H` | 512 | Output height (divisible by 64) |
| `--width`, `-W` | 512 | Output width (divisible by 64) |
| `--num-frames`, `-n` | 100 | Number of frames |
| `--seed`, `-s` | 42 | Random seed for reproducibility |
| `--fps` | 24 | Frames per second |
| `--output`, `-o` | output.mp4 | Output video path |
| `--save-frames` | false | Save individual frames as images |
| `--model-repo` | Lightricks/LTX-2 | HuggingFace model repository |

## Requirements

- **macOS with Apple Silicon** (M1, M2, M3, etc.)
- **Python** >= 3.11
- **MLX** >= 0.22.0

## Model Specifications

- **Transformer**: 48 layers, 32 attention heads, 128 dim per head
- **Latent channels**: 128
- **Text encoder**: Gemma 3 with 3840-dim output
- **RoPE**: Split mode with double precision

## Project Structure

```
mlx_video/
├── generate.py             # Video generation pipeline
├── convert.py              # Weight conversion (PyTorch → MLX)
├── postprocess.py          # Video post-processing utilities
├── utils.py                # Helper functions
└── models/
    └── ltx/
        ├── ltx.py          # Main LTXModel (DiT transformer)
        ├── config.py       # Model configuration
        ├── transformer.py  # Transformer blocks
        ├── attention.py    # Multi-head attention with RoPE
        ├── text_encoder.py # Text encoder
        ├── upsampler.py    # 2x spatial upsampler
        └── video_vae/      # VAE encoder/decoder
```

## Key Features

- **Apple Silicon Optimized** - Leverage MLX for native performance
- **Two-Stage Pipeline** - Quality refinement through upsampling
- **Reproducible** - Seed control for consistent outputs
- **Flexible Resolution** - Customize width/height/frame count
- **Full Fine-tuning** - Not just inference

## Current Status

Currently, only the distilled variant is supported. Full LTX-2 feature support is coming soon.

## License

MIT

---

**Why it's valuable for Smaug**: Good reference for video-related bookmarks that might need transcription or content extraction. Also demonstrates state-of-the-art ML infrastructure optimized for resource-constrained environments.
