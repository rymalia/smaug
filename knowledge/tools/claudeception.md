---
title: "Claudeception"
repo: "blader/Claudeception"
url: "https://github.com/blader/Claudeception"
stars: 592
language: "Shell"
category: "tool"
tags: ["Claude Code", "AI Agents", "Meta-Learning", "Skills", "Knowledge Persistence"]
---

# Claudeception

**Repository:** blader/Claudeception
**URL:** https://github.com/blader/Claudeception
**Language:** Shell
**Stars:** 592

## Description

A Claude Code skill for autonomous skill extraction and continuous learning. Have Claude Code get smarter as it works.

## Overview

Every time you use an AI coding agent, it starts from zero. You spend an hour debugging some obscure error, the agent figures it out, session ends. Next time you hit the same issue? Another hour.

Claudeception fixes this by automatically saving discovered knowledge as new skills. When Claude Code discovers something non-obvious (a debugging technique, a workaround, some project-specific pattern), it saves that knowledge as a new skill. Next time a similar problem comes up, the skill gets loaded automatically.

## Key Features

- **Automatic Skill Extraction**: Evaluates sessions for extractable knowledge
- **Semantic Matching**: Skills activate based on context matching
- **Quality Gates**: Only extracts knowledge that required actual discovery and will help with future tasks
- **Hook System**: Uses UserPromptSubmit hooks to evaluate every session
- **Meta-Skill Architecture**: Skills for acquiring skills (inspired by CASCADE research)

## How It Works

Claude Code has a native skills system that loads skill names and descriptions at startup (~100 tokens each) and matches them against current context. Claudeception writes to this retrieval system by creating new skills with descriptions optimized for future retrieval.

The key is writing good descriptions: "Helps with database problems" won't match anything useful, but "Fix for PrismaClientKnownRequestError in serverless" will match when someone hits that error.

## Installation

**User-level (recommended):**
```bash
git clone https://github.com/blader/Claudeception.git ~/.claude/skills/claudeception
```

**Project-level:**
```bash
git clone https://github.com/blader/Claudeception.git .claude/skills/claudeception
```

Then set up the activation hook in `~/.claude/settings.json`.

## Research Foundation

Based on academic work on skill libraries for AI agents:
- **Voyager** (Wang et al., 2023): Skill libraries for game-playing agents
- **CASCADE** (2024): Meta-skills (skills for acquiring skills)
- **SEAgent** (2025): Learning through trial and error
- **Reflexion** (Shinn et al., 2023): Self-reflection for agents

## Use Cases

- Debugging non-obvious errors
- Project-specific patterns and workarounds
- Configuration discoveries
- Any task where solution required meaningful discovery

## What Gets Extracted

Only extracts knowledge that:
- Required actual discovery (not just reading docs)
- Will help with future tasks
- Has clear trigger conditions
- Has been verified to work

## Source

Bookmarked from: https://x.com/blader/status/2012667150440476851
