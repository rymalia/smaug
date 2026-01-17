# compound-engineering-plugin

**Official Claude Code compound engineering plugin from Every.inc**

**Repository:** https://github.com/EveryInc/compound-engineering-plugin
**Stars:** 4858
**Language:** Python
**Last Archived:** January 14, 2026

## Overview

A Claude Code plugin that implements compound engineering methodology—making each unit of engineering work easier than the last. Philosophy: 80% planning & review, 20% execution.

## Installation

```bash
/plugin marketplace add https://github.com/EveryInc/compound-engineering-plugin
/plugin install compound-engineering
```

## Core Workflow

```
Plan → Work → Review → Compound → Repeat
```

Each cycle builds on previous learnings:
- Plans inform future plans
- Reviews catch more issues over time
- Patterns get documented and reused
- Quality stays high, so future changes remain easy

## Commands

| Command | Purpose |
|---------|---------|
| `/workflows:plan` | Turn feature ideas into detailed implementation plans |
| `/workflows:work` | Execute plans with worktrees and task tracking |
| `/workflows:review` | Multi-agent code review before merging |
| `/workflows:compound` | Document learnings to make future work easier |

## Key Concepts

### Compound Engineering Philosophy

**Traditional development accumulates technical debt.** Every feature adds complexity, making the codebase harder to work with.

**Compound engineering inverts this.** Quality and knowledge compound—each change makes the next one easier:
- Thorough planning catches issues before code
- Review captures learnings for future reference
- Documented patterns prevent re-solving problems
- Clean code keeps velocity high

### Integration with Agent Browser

The compound-engineering-plugin includes an agent-browser skill for semantic web automation within planning and work phases. This allows the plugin to:
- Interact with web-based tools and services
- Gather research during planning phase
- Verify implementations across web platforms

## Relevant Bookmarks

- **Kieran's Jan 15 Tweet:** Successfully integrated agent-browser into this plugin, replacing token-heavy alternatives

## Learn More

- [Full component reference](https://github.com/EveryInc/compound-engineering-plugin/blob/main/plugins/compound-engineering/README.md) - all agents, commands, skills
- [Compound engineering: how Every codes with agents](https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents)
- [The story behind compounding engineering](https://every.to/source-code/my-ai-had-already-fixed-the-code-before-i-saw-it)
