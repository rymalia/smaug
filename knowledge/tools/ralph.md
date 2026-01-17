---
title: "ralph"
type: tool
date_added: 2026-01-07
source: "https://github.com/snarktank/ralph"
tags: [ai-agents, autonomous, code-generation, prd, iteration, ampcode]
via: "Twitter bookmark from @ryancarson"
---

Ralph is an autonomous AI agent loop that runs Amp repeatedly until all PRD items are complete. Each iteration is a fresh Amp instance with clean context, but memory persists through git history, progress tracking, and a progress.txt file.

Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/).

## Key Features

- Autonomous iteration loop with fresh Amp instances per iteration
- Memory persistence via git history, progress.txt, and prd.json
- PRD skill for generating detailed requirements documents
- Ralph skill for converting markdown PRDs to JSON format
- Swarm mode for creating multiple parallel worktrees
- Quality checks with typecheck and test running
- Automatic commit and push on successful iterations
- Interactive flowchart for understanding the workflow

## Workflow

1. Create a PRD using the PRD skill with clarifying questions
2. Convert the markdown PRD to JSON using the Ralph skill
3. Run ralph.sh with desired max iterations
4. Ralph automatically picks highest-priority incomplete stories
5. Implements each story, runs quality checks
6. Commits if checks pass, updates prd.json
7. Appends learnings to progress.txt
8. Repeats until all stories pass or max iterations reached

## Why Bookmarked

Demonstrates a practical pattern for autonomous agent-driven development. Shows how to architect multi-iteration loops with proper memory management, quality gates, and task tracking. Valuable reference for building scalable AI-assisted development systems.

## Links

- [GitHub](https://github.com/snarktank/ralph)
- [Original Tweet](https://x.com/ryancarson/status/2009002434740601098)
