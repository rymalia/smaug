---
title: Building a Windows File Explorer from Scratch - Custom C Base Layer
author: Vjekoslav Krajačić
url: https://www.youtube.com/watch?v=bUOOaXf9qIM
type: video
category: systems-programming
tags: [C, Windows, file-explorer, performance, memory-management, custom-stdlib]
---

# Building a Windows File Explorer from Scratch

## Overview

Vjekoslav Krajačić presents how he built a Windows file explorer from scratch in C with a 2.3MB executable and no Visual C runtime dependency.

## Key Technical Decisions

### Custom Base Layer Instead of Standard Library

Rather than using the C standard library, Krajačić wrote a custom base layer because the programming landscape has evolved significantly since the standard library was designed.

### Modern Approaches Used

1. **Memory Management**
   - Memory arenas backed by VirtualAlloc
   - More efficient than malloc for batch allocations

2. **Strings**
   - Length-based strings instead of null-terminated
   - Better performance and safety properties

3. **Math Functions**
   - Custom vector and matrix implementations
   - Optimized for graphics/UI operations

### Performance Metrics

- Final executable size: 2.3MB
- No runtime dependencies
- No Visual C runtime needed

## Presentation

This talk was presented at BSC (likely a conference or event). Krajačić indicates plans to write more formal material (blog posts) about the detailed decisions and architecture.

## Relevance

Demonstrates that modern systems programming doesn't require bloated standard libraries or runtimes. Shows practical optimization techniques for Windows application development.

## Links

- **Video:** https://www.youtube.com/watch?v=bUOOaXf9qIM
- **Author:** [@vkrajacic](https://x.com/vkrajacic)
- **Related discussion:** https://x.com/shanselman/status/2009067955192054208

## Related Topics

- C systems programming
- Windows API
- Memory management strategies
- Custom standard library implementations
