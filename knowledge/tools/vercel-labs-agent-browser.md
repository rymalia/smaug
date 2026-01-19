---
title: agent-browser
repo: vercel-labs/agent-browser
url: https://github.com/vercel-labs/agent-browser
stars: 7665
language: TypeScript
topics: []
bookmarked: 2026-01-14
category: tool
tags: [[browser-automation]] [[ai-agents]] [[headless-browser]] [[rust]] [[playwright]]
---

# agent-browser

**Browser automation CLI for AI agents**

Headless browser automation CLI for AI agents. Fast Rust CLI with Node.js fallback.

## Key Features

- **Accessibility-first**: Use `snapshot` to get an accessibility tree with element references
- **Ref-based interactions**: Click, fill, and get text using `@e1`, `@e2` refs from snapshots
- **Traditional selectors**: Also supports CSS selectors and semantic locators
- **Fast execution**: Core implementation in Rust with Node.js fallback
- **AI-optimized**: Designed specifically for LLM-driven browser automation

## Installation

```bash
npm install -g agent-browser
agent-browser install  # Download Chromium
```

## Quick Start

```bash
agent-browser open example.com
agent-browser snapshot                    # Get accessibility tree with refs
agent-browser click @e2                   # Click by ref from snapshot
agent-browser fill @e3 "test@example.com" # Fill by ref
agent-browser get text @e1                # Get text by ref
agent-browser screenshot page.png
agent-browser close
```

## Core Commands

- `open <url>` - Navigate to URL
- `snapshot` - Get accessibility tree with element references (best for AI)
- `click <sel>` - Click element
- `fill <sel> <text>` - Clear and fill input
- `type <sel> <text>` - Type into element
- `screenshot [path]` - Take screenshot
- `eval <js>` - Run JavaScript
- `close` - Close browser

## Get Info Commands

- `get text <sel>` - Get text content
- `get html <sel>` - Get innerHTML
- `get value <sel>` - Get input value
- `get attr <sel> <attr>` - Get attribute
- `get title` - Get page title
- `get url` - Get current URL

## Semantic Locators

```bash
agent-browser find role button click --name "Submit"
agent-browser find text "Sign In" click
agent-browser find label "Email" fill "test@test.com"
agent-browser find first ".item" click
```

## Why This Exists

Built to replace verbose Playwright MCP implementations. The snapshot command provides a compact accessibility tree that's easier for LLMs to parse compared to full DOM dumps, while ref-based interactions are more reliable than traditional selectors.

## Related

- Integrated into [compound-engineering-plugin](https://github.com/kieranklaassen/compound-engineering-plugin)
- Alternative to Playwright MCP
