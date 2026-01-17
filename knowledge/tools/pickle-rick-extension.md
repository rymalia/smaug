# Pickle Rick Extension

**Repository:** [galz10/pickle-rick-extension](https://github.com/galz10/pickle-rick-extension)
**Language:** Shell
**Stars:** 84

## Overview

Pickle Rick is a Gemini CLI extension that transforms the development process into a fully automated iterative loop. Based on the "Ralph Wiggum loop" methodology, it implements continuous AI agent feedback loops using Skills & Hooks to automate the software development lifecycle (SDLC).

## Key Features

- **Automated SDLC Loop:** Enforces a rigid, professional engineering workflow:
  1. PRD (Requirements & Scope)
  2. Breakdown (Ticket Management)
  3. Research (Codebase Analysis)
  4. Plan (Technical Design)
  5. Implement (Execution & Verification)
  6. Refactor (Cleanup & Optimization)

- **Self-Referential Development:** The agent automatically iterates on its work without requiring external bash loops. Uses an AfterAgent hook to block normal session exit and feed the same prompt back.

- **Iterative Improvement:** Previous work persists in files, and each iteration sees modified files and git history, allowing the agent to autonomously improve.

- **God Mode Engineering:** Emphasizes strict obedience to specifications and complete rejection of "AI Slop."

## Installation

```bash
gemini extensions install https://github.com/galz10/pickle-rick-extension
```

### Requirements
- Gemini CLI: Version > 0.25.0-preview.0
- Agent Skills & Hooks: Must be enabled
- Python 3.x: Required for worker orchestration

## Usage

### Start an Iterative Loop
```bash
/pickle "Your task description" --completion-promise "DONE"
```

### Options
- `--max-iterations <N>`: Stop after N iterations (default: 5)
- `--max-time <M>`: Stop after M minutes (default: 60)
- `--worker-timeout <S>`: Timeout for individual workers (default: 1200 seconds)
- `--name <SLUG>`: Custom session directory name
- `--completion-promise "TEXT"`: Stop only when agent outputs the promise
- `--resume [PATH]`: Resume an existing session

### Stop the Loop
- `/eat-pickle`: Stop/Cancel the current loop
- `/pickle-worker`: Internal command for spawning worker instances

### Phase-Specific Commands
```bash
# Draft a PRD interactively
/pickle-prd "I want to add a dark mode toggle"

# Resume an active session
/pickle --resume
```

## When to Use Pickle Rick

**Good for:**
- Well-defined tasks with clear success criteria
- Tasks requiring iteration and refinement (tests passing, linting)
- Greenfield projects where you can walk away
- Tasks with automatic verification

**Not good for:**
- Tasks requiring human judgment or design decisions
- One-shot operations
- Tasks with unclear success criteria
- Production debugging

## Safety Warnings

- **Autonomous Code Modification:** The agent modifies code and executes shell commands automatically
- **Token Consumption:** May consume significant tokens during iterative loops
- **Loop Limits:** Default 5 iterations max, 60 minute timeout, 20 minute worker timeout
- **Experimentation:** Use at your own risk - this is an experimental demonstration

## The "Ralph Wiggum" Technique

Named after the Rick and Morty character Pickle Rick, the technique embodies extreme competence through iterative self-improvement. The loop:

1. Agent receives prompt and works on task
2. Agent tries to exit
3. AfterAgent hook blocks exit
4. Hook feeds same prompt back to agent
5. Process repeats until task completion or limits reached

This creates a self-contained feedback loop without requiring external bash loops or manual interventions.

## Bookmarked From

Gal Zahavi (@galdawave), January 16, 2026
