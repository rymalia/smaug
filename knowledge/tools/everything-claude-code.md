---
title: everything-claude-code
repo: affaan-m/everything-claude-code
url: https://github.com/affaan-m/everything-claude-code
stars: 1515
language: null
description: Complete Claude Code configuration collection - agents, skills, hooks, commands, rules, MCPs. Battle-tested configs from an Anthropic hackathon winner.
topics: []
archived: Saturday, January 17, 2026
---

# everything-claude-code

**Repository:** [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)
**Stars:** 1515
**Language:** null

## Description

Complete Claude Code configuration collection - agents, skills, hooks, commands, rules, MCPs. Battle-tested configs from an Anthropic hackathon winner.

## Overview

The complete collection of Claude Code configs from an Anthropic hackathon winner. This repo contains production-ready agents, skills, hooks, commands, rules, and MCP configurations that evolved over 10+ months of intensive use building real products.

## Key Features

### Configuration Types

- **agents/** - Specialized subagents for delegation (planner, architect, tdd-guide, code-reviewer, security-reviewer, build-error-resolver, e2e-runner, refactor-cleaner, doc-updater)
- **skills/** - Workflow definitions and domain knowledge (coding-standards, backend-patterns, frontend-patterns, project-guidelines, tdd-workflow, security-review, clickhouse-io)
- **commands/** - Slash commands for quick execution (/tdd, /plan, /e2e, /code-review, /build-fix, /refactor-clean, /test-coverage, /update-codemaps, /update-docs)
- **rules/** - Always-follow guidelines (security, coding-style, testing, git-workflow, agents, performance, patterns, hooks)
- **hooks/** - Trigger-based automations (PreToolUse, PostToolUse, Stop hooks)
- **mcp-configs/** - MCP server configurations (GitHub, Supabase, Vercel, Railway, etc.)
- **plugins/** - Plugin ecosystem documentation
- **examples/** - Example configurations (CLAUDE.md, user-CLAUDE.md, statusline.json)

## Quick Start

```bash
# Clone the repo
git clone https://github.com/affaan-m/everything-claude-code.git

# Copy agents to your Claude config
cp everything-claude-code/agents/*.md ~/.claude/agents/

# Copy rules
cp everything-claude-code/rules/*.md ~/.claude/rules/

# Copy commands
cp everything-claude-code/commands/*.md ~/.claude/commands/

# Copy skills
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

## Key Concepts

### Agents
Subagents handle delegated tasks with limited scope. Examples include planners for feature implementation, architects for system design, and reviewers for code quality.

### Skills
Reusable knowledge documents that provide context and patterns. They teach Claude domain-specific best practices without cluttering every conversation.

### Commands
Quick slash commands that trigger specific workflows. They combine agents and skills into one-line invocations.

### Rules
Global directives that apply to every session. They enforce mandatory practices like security checks and coding standards.

### Hooks
Event-driven automations that run before/after tool use or on session stop. They enable automatic workflows like documentation updates.

## Topics

None specified

## Related

- Read the full guide on X: [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- Tips, tricks and examples in author's X articles and videos

## Notes

This is a configs-only repository. The accompanying guide on X explains context window management, parallel workflows, and the philosophy behind these configurations. Evolved through intensive production use building real products over 10+ months.
