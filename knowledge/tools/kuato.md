---
title: Kuato - Session Memory for Claude Code
url: https://github.com/alexknowshtml/kuato
author: Alex Hillman
language: TypeScript
stars: 120
bookmarked: Thursday, January 8, 2026
tags: [claude-code, session-memory, ai-agents, tools]
---

# Kuato - Session Memory for Claude Code

**Repository:** https://github.com/alexknowshtml/kuato

A session recall skill for Claude Code and other coding agents that enables you to search through your conversation history and resume work from where you left off.

## Key Features

- **Zero-Setup File-Based Search:** Start immediately with text-based session searching
- **PostgreSQL Option:** Optional full-text search with faster performance on large histories
- **Session Metadata:** Tracks decisions made, files touched, tools used, and user messages
- **Token Tracking:** Monitors token usage per session and per model
- **Search Capabilities:**
  - Text search on user messages
  - File pattern filtering
  - Tool usage filtering
  - Combined queries

## Two Versions

| Feature | File-Based | PostgreSQL |
|---------|------------|-----------|
| Setup time | 0 minutes | 5 minutes |
| Dependencies | Bun only | Bun + Postgres |
| Search speed | 1-5 seconds | <100ms |
| Full-text search | Basic matching | Weighted, ranked |
| API server | No | Yes |

## Quick Start

**File-Based (No Setup):**
```bash
git clone https://github.com/alexknowshtml/kuato.git
cd kuato
bun run file-based/search.ts --query "email system" --days 7
```

**PostgreSQL:**
```bash
cd kuato/postgres
bun run db:up
bun install
DATABASE_URL="postgres://claude:sessions@localhost:5433/claude_sessions" bun run sync
DATABASE_URL="postgres://claude:sessions@localhost:5433/claude_sessions" bun run serve
```

## How It Works

Kuato doesn't require the full session transcript. It extracts:

1. **User Messages:** Tells the story of what happened
   - Requests: "Let's build an email filtering system"
   - Decisions: "Yes, use that approach"
   - Corrections: "Actually, make it async"
   - Completions: "Commit this"

2. **Files Touched:** What code was modified

3. **Tools Used:** What operations were performed

Combined, these provide enough context to resume work without reloading entire session transcripts.

## Integration with Claude Code

Use as a skill to automatically search session history when you ask questions like:
- "Where did we leave off on the email system?"
- "What decisions did we make about authentication?"
- "Show me sessions that modified components/"

## Problem Solved

Claude Code forgets everything between sessions. Kuato solves the friction of trying to remember what you were working on and the decisions you made.
