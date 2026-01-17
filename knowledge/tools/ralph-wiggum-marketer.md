---
title: "Ralph Wiggum Marketer"
type: tool
date_added: 2026-01-07
source: "https://github.com/muratcankoylan/ralph-wiggum-marketer"
language: "JavaScript"
stars: 253
tags: [ai-copywriter, claude-code, autonomous-agents, content-marketing, plugin]
via: "Twitter bookmark from @koylanai"
---

A Claude Code Plugin that provides an autonomous AI copywriter for SaaS content marketing.

Uses the Ralph Wiggum pattern - an iterative AI loop that ships content while you sleep. This plugin enables autonomous content generation with self-critique capabilities that learn from your voice and writing style.

## Key Features

- **Autonomous Copywriter Loop** - Iterative content generation with self-critique and rewriting
- **Voice Learning** - Adapts to your writing style and tone through iterative feedback
- **Multi-Agent Ecosystem** - TrendScout, Research, and Product/Marketing agents feed content pipeline
- **SQLite Content Database** - Persistent storage for trends, research, and communications
- **Task Management** - PRD-based workflow with progress tracking and learning logs
- **Git Integration** - Automatic commit and progress tracking across iterations

## Installation

### Option 1: Add as Marketplace (Recommended)

```bash
# In Claude Code, add the repo as a marketplace:
/plugin marketplace add muratcankoylan/ralph-wiggum-marketer

# Then install the plugin:
/plugin install ralph-wiggum-marketer@muratcankoylan-ralph-wiggum-marketer
```

### Option 2: Test Locally (For Development)

```bash
# Clone the repo
git clone https://github.com/muratcankoylan/ralph-wiggum-marketer.git

# Run Claude Code with the plugin directory
claude --plugin-dir ./ralph-wiggum-marketer
```

## Commands

| Command | Description |
|---------|-------------|
| `/ralph-init` | Initialize a new content project in current directory |
| `/ralph-marketer` | Start the autonomous copywriter loop |
| `/ralph-status` | Check content pipeline and progress |
| `/ralph-cancel` | Cancel the active loop |

## The Ralph Loop

1. **Read PRD** - Check project requirements and tasks
2. **Check Progress** - Review learnings and current state
3. **Pick Task** - Find highest priority story
4. **Execute** - Complete the task following acceptance criteria
5. **Verify** - Run tests to ensure quality
6. **Commit** - Save progress to git
7. **Update** - Mark task done, log learnings
8. **Repeat** - Loop until all tasks complete

## Architecture

Multi-agent ecosystem with:
- **TrendScout Agent** - Identifies relevant trends
- **Research Agent** - Gathers research and data points
- **Product/Marketing Agent** - Provides product and market context
- **Ralph the Copywriter** - Central AI that reads inputs, plans content, writes, reviews, and iterates

Each iteration is a fresh context window with memory persisting through markdown files and git commits.

## Links

- [GitHub Repository](https://github.com/muratcankoylan/ralph-wiggum-marketer)
- [Original Tweet](https://x.com/koylanai/status/2008828205801885911)
