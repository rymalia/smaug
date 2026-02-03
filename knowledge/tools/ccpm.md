---
title: "Claude Code PM: Spec-Driven Development"
date: "Monday, February 2, 2026"
url: "https://github.com/automazeio/ccpm"
author: "automaze"
repository: "automazeio/ccpm"
stars: 6446
language: "Shell"
tags: ["ai-agents", "ai-coding", "claude", "claude-code", "project-management", "spec-driven"]
---

# Claude Code PM (CCPM)

A battle-tested system that turns PRDs into epics, epics into GitHub issues, and issues into production code – with full traceability at every step.

## The Problem It Solves

Traditional development workflows suffer from:
- **Context evaporation** between sessions, forcing constant re-discovery
- **Parallel work conflicts** when multiple developers touch the same code
- **Requirements drift** as verbal decisions override written specs
- **Invisible progress** until the very end

## The Solution

CCPM uses **spec-driven development** with **GitHub Issues as the database**, enabling:

### True Team Collaboration
- Multiple Claude instances can work on the same project simultaneously
- Human developers see AI progress in real-time through issue comments
- Team members can jump in anywhere – context is always visible
- Managers get transparency without interrupting flow

### Seamless Human-AI Handoffs
- AI can start a task, humans can finish it (or vice versa)
- Progress updates visible to everyone, not trapped in chat logs
- Code reviews happen naturally through PR comments

### Single Source of Truth
- No separate databases or project management tools
- Issue state = project state
- Comments = audit trail
- Labels = organization

## The Workflow

```
PRD Creation → Epic Planning → Task Decomposition → GitHub Sync → Parallel Execution
```

## Core Principle: No Vibe Coding

Every line of code has a spec behind it. Development is driven by written requirements, not intuition.

## Quick Start

```bash
# Create a comprehensive PRD through guided brainstorming
/pm:prd-new memory-system

# Transform PRD into a technical epic with task breakdown
/pm:prd-parse memory-system

# Push to GitHub and start parallel execution
/pm:epic-oneshot memory-system
/pm:issue-start 1235
```

## Key Features

- **Parallel agent execution** on independent tasks using Git worktrees
- **GitHub Issues integration** for persistent, transparent tracking
- **Intelligent prioritization** with `/pm:next`
- **Full audit trail** through issue comments
- **CI/CD integration** for automated validation

## What Makes It Different

| Traditional | Claude Code PM |
|-------------|-----------------|
| Context lost between sessions | Persistent context across all work |
| Serial task execution | Parallel agents on independent tasks |
| "Vibe coding" from memory | Spec-driven with full traceability |
| Progress hidden in branches | Transparent audit trail in GitHub |
| Manual task coordination | Intelligent prioritization |

## Documentation

- [Getting Started](https://github.com/automazeio/ccpm#get-started-now)
- [Full Documentation](https://github.com/automazeio/ccpm)
- [Technical Architecture](https://github.com/automazeio/ccpm#system-architecture)

---

**Bookmarked from:** @dr_cintas tweet demonstrating automated pipeline from PRD to production code
