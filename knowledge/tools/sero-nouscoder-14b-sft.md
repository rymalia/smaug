---
title: Sero NousCoder 14B SFT
author: 0xSero
url: https://huggingface.co/0xSero/sero-nouscoder-14b-sft
model: NousCoder-14B
type: Fine-tuned Language Model
status: active
source: twitter-bookmark
---

# Sero NousCoder 14B SFT

A fine-tuned version of NousCoder 14B trained on personal AI assistant chat history across multiple platforms.

## Overview

This model was created by:
1. Extracting all chat history from multiple AI coding assistants (Claude Code, Cursor, Codex, etc.)
2. Processing and preparing the data using the AI Data Extraction Toolkit
3. Fine-tuning NousCoder 14B with LoRA (Low-Rank Adaptation) on this personal data

## Capabilities

- Code generation and completion tailored to personal coding style
- Understanding of personal preferences and patterns from AI assistant interactions
- Context-aware suggestions based on historical patterns
- Optimized for the author's typical workflows across different projects

## Training Data

Trained on authentic, multi-turn conversations including:
- Code context and snippets
- Problem-solving approaches
- Architectural discussions
- Debugging sessions
- Code reviews and suggestions

## Use Cases

- Personal code completion assistant
- Faster context-aware coding
- Pattern-based code generation matching personal style
- Research into AI-assisted development workflows

## Technical Details

- **Base Model:** NousCoder-14B
- **Fine-tuning Method:** LoRA (Low-Rank Adaptation)
- **Training Data:** Multi-tool AI assistant conversations
- **Format:** Hugging Face Model Hub compatible

## Model Hub

Available on Hugging Face: [0xSero/sero-nouscoder-14b-sft](https://huggingface.co/0xSero/sero-nouscoder-14b-sft)
