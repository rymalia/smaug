---
title: SimpleLLM - Minimal LLM Inference Engine
url: https://github.com/naklecha/simple-llm
stars: 158
language: Python
archived: false
---

# SimpleLLM

A ~950 line, minimal, extensible LLM inference engine built from scratch for educational purposes and research experimentation.

## Overview

SimpleLLM is a working implementation of current state-of-the-art inference techniques in approximately 760 lines of code. It's designed to be read, extended, and modified by researchers wanting to understand how modern LLM inference engines work.

## Performance

Comparable performance to vLLM on single H100 GPU with max_tokens=1000:

- **Batch size 1:** 135 tok/s (vLLM: 138 tok/s)
- **Batch size 64:** 4,041 tok/s (vLLM: 3,846 tok/s)

## Key Components

| Component | Lines |
|-----------|-------|
| `llm.py` (engine) | 563 |
| `model/model.py` | 324 |
| `model/tokenizer.py` | 92 |

## Modern Inference Techniques Included

- **Async by default** - Background inference loop with request queue
- **Continuous batching** - New requests join mid-generation without waiting
- **CUDA graphs** - Eliminates kernel launch overhead for decode steps
- **Slot-based KV cache** - Zero-copy sequence management
- **Fused operations** - QKV projections, RMSNorm + residual, RoPE
- **Flash Attention 2** - Memory-efficient attention for prefill and decode
- **Paged KV cache** - Pre-allocated memory pages for efficient growth
- **GQA (Grouped Query Attention)** - 8 KV heads shared across 64 query heads

## Use Cases

**For researchers:** Simple but performant starting point for experimenting with novel inference techniques without wading through complex production code.

**For research labs:** Pre-stripped-down alternative to forking vLLM, suitable for implementing custom kernels or infrastructure adaptations.

**For students:** Learn how inference engines work by tracing through the entire request lifecycle in readable code.

## Current Limitations

- Only supports `OpenAI/gpt-oss-120b` on single NVIDIA H100
- Chosen to demonstrate viability at complex starting point (MoE + large model)

## Future Roadmap

- Paged attention
- Tensor parallelism on 8x H100s
- Support for other MoE models
- Support for other architectures

## Resources

- Repository: https://github.com/naklecha/simple-llm
- License: Apache 2.0
