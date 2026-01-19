---
title: compound-engineering-plugin
repo: kieranklaassen/compound-engineering-plugin
url: https://github.com/kieranklaassen/compound-engineering-plugin
stars: 4953
language: Python
topics: []
bookmarked: 2026-01-14
category: tool
tags: [[claude-code]] [[plugin]] [[engineering-workflow]] [[code-review]] [[planning]] [[ai-agents]]
---

# compound-engineering-plugin

**Official Claude Code compound engineering plugin**

A Claude Code plugin marketplace featuring the **Compound Engineering Plugin** — tools that make each unit of engineering work easier than the last.

## Philosophy

**Each unit of engineering work should make subsequent units easier—not harder.**

Traditional development accumulates technical debt. Every feature adds complexity. The codebase becomes harder to work with over time.

Compound engineering inverts this. 80% is in planning and review, 20% is in execution:
- Plan thoroughly before writing code
- Review to catch issues and capture learnings
- Codify knowledge so it's reusable
- Keep quality high so future changes are easy

## Installation

```bash
/plugin marketplace add https://github.com/kieranklaassen/compound-engineering-plugin
/plugin install compound-engineering
```

## Workflow

```
Plan → Work → Review → Compound → Repeat
```

| Command | Purpose |
|---------|---------|
| `/workflows:plan` | Turn feature ideas into detailed implementation plans |
| `/workflows:work` | Execute plans with worktrees and task tracking |
| `/workflows:review` | Multi-agent code review before merging |
| `/workflows:compound` | Document learnings to make future work easier |

Each cycle compounds: plans inform future plans, reviews catch more issues, patterns get documented.

## Key Features

- **Detailed Planning**: Turn vague ideas into concrete implementation plans
- **Worktree Management**: Execute work in isolated git worktrees
- **Multi-Agent Review**: Comprehensive code review from multiple perspectives
- **Knowledge Capture**: Document learnings to improve future work
- **Quality Focus**: Maintain high standards to prevent technical debt

## Skills Included

- **agent-browser**: Browser automation for AI agents (replacing Playwright MCP)
- Workflow automation
- Code review agents
- Planning assistants

## Related Resources

- [Compound engineering: how Every codes with agents](https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents)
- [The story behind compounding engineering](https://every.to/source-code/my-ai-had-already-fixed-the-code-before-i-saw-it)

## Why It Matters

Instead of accumulating technical debt where each change makes the next harder, compound engineering ensures each change makes the next easier. Quality compounds, velocity increases, and the codebase becomes more maintainable over time.
