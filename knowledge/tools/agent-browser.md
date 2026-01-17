# agent-browser

**Headless browser automation CLI for AI agents**

**Repository:** https://github.com/vercel-labs/agent-browser
**Stars:** 7027
**Language:** TypeScript
**Last Archived:** January 14, 2026

## Overview

Fast Rust CLI with Node.js fallback for automating browser interactions. Designed specifically for AI agents with semantic locators and accessibility tree snapshots.

## Installation

### npm (recommended)

```bash
npm install -g agent-browser
agent-browser install  # Download Chromium
```

### From Source

```bash
git clone https://github.com/vercel-labs/agent-browser
cd agent-browser
pnpm install
pnpm build
pnpm build:native   # Requires Rust (https://rustup.rs)
pnpm link --global  # Makes agent-browser available globally
agent-browser install
```

### Linux Dependencies

On Linux, install system dependencies:

```bash
agent-browser install --with-deps
# or manually: npx playwright install-deps chromium
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

### Traditional Selectors (also supported)

```bash
agent-browser click "#submit"
agent-browser fill "#email" "test@example.com"
agent-browser find role button click --name "Submit"
```

## Key Commands

### Core Navigation & Interaction

- `open <url>` - Navigate to URL (aliases: goto, navigate)
- `click <sel>` - Click element
- `dblclick <sel>` - Double-click element
- `type <sel> <text>` - Type into element
- `fill <sel> <text>` - Clear and fill field
- `hover <sel>` - Hover element
- `select <sel> <val>` - Select dropdown option
- `screenshot [path]` - Take screenshot (--full for full page)
- `snapshot` - Get accessibility tree with semantic refs (best for AI)

### Get Info

- `get text <sel>` - Get text content
- `get html <sel>` - Get innerHTML
- `get value <sel>` - Get input value
- `get url` - Get current page URL
- `get title` - Get page title

### Semantic Locators (AI-friendly)

```bash
agent-browser find role button click --name "Submit"
agent-browser find text "Sign In" click
agent-browser find label "Email" fill "test@test.com"
agent-browser find placeholder "Search..." fill "query"
```

**Supported roles:** button, link, textbox, checkbox, radio, etc.
**Actions:** click, fill, check, hover, text

### Wait States

```bash
agent-browser wait <selector>              # Wait for element
agent-browser wait 1000                    # Wait 1 second
agent-browser wait --text "Welcome"        # Wait for text
agent-browser wait --url "**/dash"         # Wait for URL pattern
agent-browser wait --load networkidle      # Wait for network idle
```

## Why It's Better for Agents

1. **Semantic Locators** - Find elements by ARIA role, text, label, not brittle CSS selectors
2. **Accessibility Tree** - `snapshot` command returns refs like `@e1`, `@e2` for quick reference
3. **Token Efficient** - Structured output instead of raw HTML dumps
4. **Fast** - Rust implementation with Node fallback for cross-platform support
5. **No 12k Token Overhead** - Dynamic loading reduces context window bloat compared to Playwright MCP

## Relevant Bookmarks

- **Kieran's Jan 14 Tweet:** Successfully replaced Playwright MCP in compound engineering plugin
- **Kieran's Follow-up:** Agent Browser integration working well, reduced token usage significantly
