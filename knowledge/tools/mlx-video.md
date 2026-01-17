# MLX-Video

**Repository:** [Blaizzy/mlx-video](https://github.com/Blaizzy/mlx-video)
**Language:** Python
**Stars:** 47

## Overview

MLX-Video is a package for inference and finetuning of image-video-audio generation models optimized for Mac using MLX (Apple's machine learning framework). It provides the best performance for generative AI models on Apple Silicon devices.

## Supported Models

### LTX-2
- **Parameters:** 19B (distilled variant currently supported)
- **From:** Lightricks
- **Model Link:** [Lightricks/LTX-Video on Hugging Face](https://huggingface.co/Lightricks/LTX-Video)

## Features

- **Text-to-Video Generation:** Create videos from text descriptions
- **Two-Stage Generation Pipeline:** High-quality output through staged processing
  1. Generate at half resolution with 8 denoising steps
  2. 2x spatial upsampling via LatentUpsampler
  3. Refine at full resolution with 3 denoising steps
  4. VAE decoder converts latents to RGB video
- **Spatial Upscaling:** 2x upscaling for images and videos
- **Apple Silicon Optimized:** Leverages MLX for efficient inference on Macs

## Installation

### Option 1: With pip (requires git)
```bash
pip install git+https://github.com/Blaizzy/mlx-video.git
```

### Option 2: With uv (ultra-fast package manager)
```bash
uv pip install git+https://github.com/Blaizzy/mlx-video.git
```

## Usage

### Text-to-Video Generation

Basic usage:
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
| `--height`, `-H` | 512 | Output height (must be divisible by 64) |
| `--width`, `-W` | 512 | Output width (must be divisible by 64) |
| `--num-frames`, `-n` | 100 | Number of frames |
| `--seed`, `-s` | 42 | Random seed for reproducibility |
| `--fps` | 24 | Frames per second |
| `--output`, `-o` | output.mp4 | Output video path |
| `--save-frames` | false | Save individual frames as images |
| `--model-repo` | Lightricks/LTX-2 | HuggingFace model repository |

## Requirements

- **OS:** macOS with Apple Silicon
- **Python:** >= 3.11
- **MLX:** >= 0.22.0

## Model Specifications

- **Architecture:** DiT (Diffusion Transformer)
- **Transformer Layers:** 48
- **Attention Heads:** 32
- **Dimension Per Head:** 128
- **Latent Channels:** 128
- **Text Encoder:** Gemma 3 with 3840-dim output
- **RoPE Mode:** Split mode with double precision

## Project Structure

```
mlx_video/
├── generate.py             # Video generation pipeline
├── convert.py              # Weight conversion (PyTorch -> MLX)
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

## Status

Currently supports the distilled LTX-2 variant. Full LTX-2 feature support is coming soon.

## License

MIT

## Bookmarked From

Prince Canuma (@Prince_Canuma), January 16, 2026
