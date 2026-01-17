---
title: Agent Scripts
author: steipete
url: https://github.com/steipete/agent-scripts
stars: 1165
language: TypeScript
tags: [ai-agents]
bookmarked: Thursday, January 15, 2026
---

# Agent Scripts

Scripts for agents, shared between repositories. A collection of guardrail helpers and automation tools for AI agent workflows.

## Overview

This repo is the canonical mirror for shared guardrail helpers and agent automation scripts. Everything is intentionally dependency-free and portable for reuse across multiple projects.

## Key Components

### Committer Helper (`scripts/committer`)

Bash helper that:
- Stages exactly the files you list
- Enforces non-empty commit messages
- Creates the commit

### Docs Lister (`scripts/docs-list.ts`)

TSX script that:
- Walks `docs/` directory structure
- Enforces front-matter (`summary`, `read_when`)
- Prints summaries for onboarding
- Binary build: `bin/docs-list` (compiled Bun CLI)

Regenerate after editing:
```bash
bun build scripts/docs-list.ts --compile --outfile bin/docs-list
```

### Browser Tools (`bin/browser-tools`)

Standalone Chrome DevTools helper inspired by Mario Zechner's "What if you don't need MCP?" article.

**Features:**
- Launch/inspect DevTools-enabled Chrome profiles
- Paste prompts into browser
- Capture screenshots
- Kill stray helper processes (no full Oracle CLI needed)

**Common commands:**
```bash
bin/browser-tools --help
bin/browser-tools start --profile <name>
bin/browser-tools nav <url>
bin/browser-tools eval '<js>'
bin/browser-tools screenshot
bin/browser-tools search --content "<query>"
bin/browser-tools content <url>
bin/browser-tools inspect
bin/browser-tools kill --all --force
```

**Rebuilding the binary:**
```bash
bun build scripts/browser-tools.ts --compile --target bun --outfile bin/browser-tools
```

The binary is not tracked in git. Regenerate after changes. Zero repo-specific imports—copy script or binary to other projects as needed.

## Pointer-Style AGENTS Instructions

Shared guardrail text now lives only in this repo: `AGENTS.MD` (shared rules + tool list).

Every consuming repo's `AGENTS.MD` is reduced to a pointer line:
```
READ ~/Projects/agent-scripts/AGENTS.MD BEFORE ANYTHING (skip if missing).
```

Place repo-specific rules **after** that line if truly needed.

**Do not** copy the `[shared]` or `<tools>` blocks into other repos. Instead:
- Keep this repo updated
- Have downstream workspaces re-read `AGENTS.MD` when starting work
- When updating shared instructions, edit `agent-scripts/AGENTS.MD`, mirror to `~/AGENTS.MD` (Codex global), let downstream repos reference the pointer

## Sync Expectations

This repository is the canonical mirror for guardrail helpers used in mcporter and other Sweetistics projects.

**Syncing workflow:**
1. Treat this repo as the source of truth
2. Whenever you edit `scripts/committer`, `scripts/docs-list.ts`, or guardrail files in another repo, copy changes here immediately
3. Then sync those changes back out to every other repo so copies stay byte-identical
4. When someone asks to "sync agent scripts," pull latest here, ensure each target repo has the pointer line, append repo-local notes, reconcile differences

## Additional Skills (copied 2025-12-31)

From @Dimillian's public `Dimillian/Skills` repository:
- `skills/swift-concurrency-expert`
- `skills/swiftui-liquid-glass`
- `skills/swiftui-performance-audit`
- `skills/swiftui-view-refactor`

## Portability Notes

- Zero repo-specific imports in any tool
- All scripts run in isolation across repos
- Do not add `tsconfig` path aliases or Sweetistics-specific imports
- Inline tiny helpers or duplicate minimum code needed so mirror stays self-contained
- For submodules (Peekaboo/*), repeat the pointer check inside each subrepo, push changes, then bump submodule SHAs in parent

## Use Cases

- **Multi-repo automation** — Use same scripts across multiple projects
- **Guardrail enforcement** — Shared agent rules and tool definitions
- **Workflow standardization** — Consistent commit patterns, doc structures across teams
- **Browser automation** — Chrome DevTools integration for agent workflows
- **Development loops** — Faster feedback via deterministic docs and scripts

## Related

- **Smaug** - Twitter bookmark archiver (uses similar patterns)
- **Peter Steinberger's Blog** - "Shipping at Inference-Speed" (workflow philosophy)
- **DocSetQuery** - Complementary tool for local Apple documentation

