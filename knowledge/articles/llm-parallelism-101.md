# LLM Parallelism 101: A Complete Systems Guide

**Author:** Ahmad Osman (@TheAhmadOsman)
**Source:** https://x.com/TheAhmadOsman/status/2012467123050725726
**Date:** January 17, 2026

## Overview

Comprehensive deep-dive into parallelism strategies for large language models, breaking down how to split trillion-parameter models across thousands of GPUs.

## Core Concepts Covered

### What Gets Split?

- **Weights** - Model parameters distributed across devices
- **Computation** - Matrix multiplies and operations parallelized
- **State** - Activations, KV cache, optimizer states sharded
- **Data** - Training batches divided across GPUs
- **Sequence** - Token positions split for long contexts

### Five Parallelism Axes

#### 1. Data Parallelism (DP)
- Each GPU holds full model copy
- Different batch slices on each device
- All-reduce for gradient synchronization
- Modern: ZeRO/FSDP shards optimizer states, gradients, params

#### 2. Tensor Parallelism (TP)
- Shard large matrix multiplies within layers
- Split attention heads across GPUs
- Minimal latency with NVLink (hidden in GEMM)
- First choice when model won't fit one GPU

#### 3. Pipeline Parallelism (PP)
- "Assembly line" - split model by layers
- Each stage on different GPU/node
- Microbatching reduces pipeline bubbles
- 1F1B (one-forward-one-backward) scheduling

#### 4. Expert Parallelism (EP / Mixture of Experts)
- Route tokens to subset of experts
- Only active parameters per token
- Scale parameters without compute scaling
- All-to-all communication overhead is main cost

#### 5. Sequence/Context Parallelism (SP/CP)
- Split along sequence axis, not layers
- Essential for 128k+ token contexts
- Megatron CP, DeepSpeed-Ulysses, Ring-Attention
- Solves KV cache memory bottleneck

## Inference vs Training

### Inference Characteristics
- Cares about context length
- KV cache size is critical
- Throughput from batching many requests
- Prefill (prompt encoding) vs decode (one token) are different bottlenecks

### Training Characteristics
- Wants large batches for efficiency
- Needs fast gradient synchronization
- Activations consume massive memory
- All-reduce operations are dominant overhead

## Hardware Reality

- 95% of compute is big matrix multiplies (GEMMs)
- Memory bandwidth is the actual ceiling, not FLOPs
- Interconnect speed (NVLink, Infiniband) is critical
- Collective operations (all-reduce, all-to-all) are expensive

## Real-World Combinations

### Serving (Inference)
- Model fits 1 GPU? → Data parallelism for batching
- Model too big? → Add tensor parallelism (2-4 GPUs intra-node)
- Still too big? → Pipeline parallelism (split layers)
- Long contexts? → Context/sequence parallelism + paged attention
- MoE? → Expert parallelism with top-1 routing

### Training
- Start with data parallelism + ZeRO/FSDP
- Model too wide? → Add tensor parallelism
- Model too deep? → Add pipeline parallelism (interleaved)
- MoE? → 3D/4D grid: TP × EP × DP × PP
- Huge contexts? → Sequence parallelism + activation checkpointing

## Modern Production Tricks

- **Paged attention** - Manage KV cache like OS virtual memory
- **Continuous batching** - vLLM, TensorRT-LLM style serving
- **GQA/MQA** - Grouped/multi-query attention shrinks KV footprint
- **Context parallelism** - Finally scales prefill to 128k+ tokens
- **Interleaved pipelines** - ~10% throughput gain, nearly free
- **Expert-Choice routing** - Better load balancing than top-K

## Key Rules of Thumb

1. **TP is your friend** - Split big layers, keep intra-node with fast links
2. **Data parallelism for throughput** - Add ZeRO for memory relief
3. **Pipeline for tall models** - Necessary for true giants, adds latency
4. **Expert parallelism unlocks scale** - But watch your network
5. **Context parallelism for long prompts** - Essential for 128k+ contexts
6. **Combine axes for bottleneck** - 3D/4D/5D grids, not single axes

## The Systems View

```
Bottleneck → Solution
├─ VRAM full → Reduce batch, use DP/ZeRO, add TP
├─ Weights too big → TP or PP
├─ KV cache too big → CP/SP or GQA/MQA
├─ Single token latency → TP (keep intra-node)
├─ Throughput too low → DP or reduce model precision
└─ Gradient sync slow → Use ring all-reduce or FSDP
```

## Takeaways

- Every parallelism axis is a lever, not a panacea
- Real skill is matching your exact bottleneck to the right split
- "8M tokens/sec on 1024 GPUs" means understanding how they split the problem
- Hardware-aware optimization is survival for large models
- Combine parallelism axes strategically for your specific constraints
