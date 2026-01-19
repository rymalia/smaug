---
title: "getspecstory"
url: https://github.com/specstoryai/getspecstory
category: tool
tags: [ai, chat-history, claude, claude-code, cursor, copilot, vscode, session-management]
date: 2026-01-16
stars: 961
language: Go
---

# getspecstory

**SpecStory CLI - Normalize AI agent sessions across tools**

## Overview
SpecStory is an open-source CLI that normalizes AI agent sessions and defines a stable contract for AI conversations across different development tools. It captures, indexes, and makes searchable every interaction with AI coding assistants across all projects and tools.

## Key Features
- **Local-first storage**: Automatically saves AI interactions to `.specstory/history/` in your project
- **Multi-tool support**: Works with Claude Code, Cursor, Copilot, Codex CLI, Gemini CLI
- **Cloud sync (optional)**: Only syncs if logged in, keeps everything local otherwise
- **Search capabilities**: Find conversations locally or across all projects in cloud
- **Provenance tracking**: Maintains stable contract for AI conversations

## Supported Tools
| Product | Type | Agents Supported |
|---------|------|-----------------|
| Cursor Extension | IDE | Cursor AI |
| VSC Copilot Extension | IDE | GitHub Copilot |
| SpecStory CLI | CLI | Claude Code, Cursor CLI, Codex CLI, Gemini CLI |

## How It Works
```
AI Coding Tools              Local First                  Cloud Platform
─────────────────           ─────────────                ─────────────────
                                                          (Login Required)
Cursor IDE         ┐
Copilot IDE        │
Claude Code CLI    ├──────►  .specstory/history/  ──────►  cloud.specstory.com
Cursor CLI         │          (Auto-Saved Locally)        (Search, Ask & Share)
Codex CLI          │
Gemini CLI         ┘
```

## Workflow
1. **Capture** - Extensions save every AI interaction locally to `.specstory/history/`
2. **Sync (Optional)** - Only if logged in, sessions sync to cloud
3. **Search** - Find conversations locally or across all projects in cloud
4. **Share** - Export and share specific solutions with your team

## Installation
```bash
brew tap specstoryai/tap
brew install specstory
```

## Why It Matters
Turn AI development conversations into searchable, shareable knowledge. Never lose a brilliant solution, code snippet, or architectural decision again. The tagline: "Intent is the new source code."

## Links
- **Repository:** https://github.com/specstoryai/getspecstory
- **Stars:** 961
- **Language:** Go
- **Topics:** ai, chat-history, claude, claude-code, codex, copilot, cursor-ai, vscode

## Related
Similar to Kuato session memory system, but with cross-tool support and cloud sync capabilities.
