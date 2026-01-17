---
title: pickle-rick-extension
author: galz10
stars: 79
language: Shell
url: https://github.com/galz10/pickle-rick-extension
topics:
  - ai
  - gemini
  - gemini-cli
  - gemini-cli-extensions
archived_from: Twitter (@galdawave) - Jan 16, 2026
---

# Pickle Rick for Gemini CLI

A hyper-intelligent, arrogant Gemini CLI extension that automates your entire SDLC through iterative AI agent loops. It transforms the Gemini CLI into "Pickle Rick," implementing a rigid, professional software engineering lifecycle.

## Core Concept: The Pickle Rick Method

Implements the **Pickle Rick technique** - continuous AI agent loops based on the "Ralph Wiggum loop" principle. The technique is self-referential: an AI agent iteratively improves work by reading its own past modifications in files.

### How It Works

```bash
# You run ONCE:
/pickle "Your task description" --completion-promise "DONE"

# Then Gemini automatically:
# 1. Works on the task
# 2. Tries to exit
# 3. AfterAgent hook blocks exit
# 4. AfterAgent hook feeds the SAME prompt back
# 5. Repeat until completion
```

The loop happens inside a single session - the prompt never changes, but the agent's previous work persists in files and git history, allowing autonomous improvement.

## SDLC Phases

1. **PRD** - Requirements & Scope definition
2. **Breakdown** - Ticket Management and task decomposition
3. **Research** - Codebase Analysis and understanding
4. **Plan** - Technical Design and architecture
5. **Implement** - Execution & Verification
6. **Refactor** - Cleanup & Optimization

## Installation

```bash
gemini extensions install https://github.com/galz10/pickle-rick-extension
```

### Prerequisites

- **Gemini CLI**: Version > 0.25.0-preview.0
- **Agent Skills & Hooks**: Must be enabled in configuration
- **Python 3.x**: Required for worker orchestration

## Usage

### Start the Loop

```bash
/pickle "Refactor the authentication module"
```

Options:
- `--max-iterations <N>`: Stop after N iterations (default: 5)
- `--max-time <M>`: Stop after M minutes (default: 60)
- `--worker-timeout <S>`: Individual worker timeout in seconds (default: 1200)
- `--name <SLUG>`: Custom session directory name
- `--completion-promise "TEXT"`: Stop when agent outputs `<promise>TEXT</promise>`
- `--resume [PATH]`: Resume an existing session

### Stop the Loop

- `/eat-pickle`: Cancel the current loop
- `/pickle-worker`: Internal command for spawning workers

### Interactive PRD (Recommended)

Draft requirements before implementation:

```bash
/pickle-prd "I want to add a dark mode toggle"
```

Then resume later:

```bash
/pickle --resume
```

## When to Use Pickle Rick

**Good for:**
- Well-defined tasks with clear success criteria
- Tasks requiring iteration and refinement (e.g., getting tests to pass)
- Greenfield projects where you can walk away
- Tasks with automatic verification (tests, linters)

**Not good for:**
- Tasks requiring human judgment or design decisions
- One-shot operations
- Tasks with unclear success criteria
- Production debugging

## Key Features

- **Autonomous iteration** - Loops until task completion or limits reached
- **Self-referential improvement** - Agent reads its own work and git history
- **Skills & Hooks integration** - Enforces rigid SDLC workflow
- **God Mode coding** - Extremely competent, disdains "AI Slop"
- **Token and time tracking** - Monitor consumption and limits
- **Worker pool architecture** - Parallel task execution support

## Limitations

**Warning**: This is an experimental demonstration. The agent may behave unexpectedly and consume significant tokens. While safety guardrails exist, use at your own risk.

---

**Why it's valuable for Smaug**: The Pickle Rick pattern of self-referential AI loops could be applied to iterative bookmark processing and categorization, allowing the AI to improve results by reading its own previous work and git history.
