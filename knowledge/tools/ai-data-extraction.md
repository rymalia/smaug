---
title: AI Data Extraction Toolkit
author: 0xSero
url: https://github.com/0xSero/ai-data-extraction
stars: 148
language: Python
status: active
source: twitter-bookmark
---

# AI Coding Assistant Training Data Extraction Toolkit

Complete toolkit to extract ALL chat, agent, and code context data from AI coding assistants for machine learning training.

## What This Does

Automatically discovers and extracts **complete conversation history** including:
- User messages & AI responses
- Code context (file paths, line numbers, snippets)
- Code diffs and suggested edits
- Multi-file contexts
- Tool use and execution results
- Timestamps and metadata

## Supported Tools

1. **Claude Code / Claude Desktop** - Extracts from `~/.claude` and related directories
2. **Codex** - Extracts from `~/.codex` directories
3. **Cursor** - All versions (Chat, Composer, Agent modes)
4. **Trae** - Extracts from `~/.trae` and application support
5. **Windsurf** - Extracts from application support directories
6. **Continue AI** - Extracts from `~/.continue/sessions/`
7. **Google Gemini CLI** - Extracts from `~/.gemini/tmp/` with thoughts and reasoning

## Key Features

- **Auto-discovery** - Finds all installations across common OS locations
- **SQLite parsing** - Handles VSCode-like database formats
- **JSONL extraction** - Processes session files
- **Complete context** - Captures code snippets, diffs, and execution results
- **Cross-platform** - Works on macOS, Linux, and Windows
- **No dependencies** - Uses Python 3 standard library only

## Output Format

Conversations saved as JSONL (one JSON object per line) with:
- Message history with roles
- Code context with file paths and line ranges
- Suggested diffs and edits
- Timestamps and model information
- Metadata about source tool

## Use Cases

- Train custom language models on your coding patterns
- Analyze your interaction patterns with AI assistants
- Create personalized code completion models
- Research AI-assisted development workflows

## Repository

[0xSero/ai-data-extraction on GitHub](https://github.com/0xSero/ai-data-extraction)
