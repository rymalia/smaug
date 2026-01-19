---
title: dotagents
url: https://github.com/iannuttall/dotagents
author: iannuttall
stars: 325
language: TypeScript
category: tool
tags: [ai-tools, configuration-management, symlinks, claude, cursor, codex, factory, opencode, developer-tools]
archived: 2026-01-18
---

# dotagents

One canonical `.agents` folder that powers all your AI tools.

## Overview

dotagents solves the problem of managing duplicate configurations across multiple AI coding assistants (Claude, Factory, Codex, Cursor, OpenCode). Instead of maintaining separate `.claude/`, `.factory/`, `.cursor/`, etc. directories with duplicated commands, hooks, and skills, dotagents creates a single `.agents/` folder as the source of truth and symlinks everything else to it.

## Key Features

- **Single source of truth**: All commands, hooks, skills, and configuration files live in `.agents/`
- **Multi-client support**: Works with Claude, Factory, Codex, Cursor, and OpenCode
- **Safe operations**: Always creates backups before overwriting
- **Flexible scope**: Choose global (home directory) or project-specific setup
- **Reversible**: Built-in undo functionality to restore previous state
- **Interactive CLI**: Guided setup with client selection

## Quick Start

Requirements: Bun 1.3+

Run the guided CLI:
```bash
npx @iannuttall/dotagents
# or with Bun:
bunx @iannuttall/dotagents
```

Choose a workspace (Global home or Project folder), select the clients you want to manage, and follow the prompts. You can run it again anytime to repair links or undo changes.

## What It Does

- Keeps `.agents` as the source of truth
- Creates symlinks for Claude, Codex, Factory, Cursor, and OpenCode (based on your selection)
- Always creates a backup before any overwrite so changes are reversible

## Where It Links (Global Scope)

- `.agents/CLAUDE.md` → `~/.claude/CLAUDE.md` (if present)
- `.agents/AGENTS.md` → `~/.claude/CLAUDE.md` (fallback when no CLAUDE.md)
- `.agents/commands` → `~/.claude/commands`
- `.agents/commands` → `~/.factory/commands`
- `.agents/commands` → `~/.codex/prompts`
- `.agents/commands` → `~/.cursor/commands`
- `.agents/hooks` → `~/.claude/hooks`
- `.agents/hooks` → `~/.factory/hooks`
- `.agents/AGENTS.md` → `~/.factory/AGENTS.md`
- `.agents/AGENTS.md` → `~/.codex/AGENTS.md`
- `.agents/AGENTS.md` → `~/.config/opencode/AGENTS.md`
- `.agents/commands` → `~/.opencode/commands`
- `.agents/skills` → `~/.claude/skills`
- `.agents/skills` → `~/.factory/skills`
- `.agents/skills` → `~/.codex/skills`
- `.agents/skills` → `~/.cursor/skills`
- `.agents/skills` → `~/.opencode/skills`

Project scope links only commands/hooks/skills into the project's client folders (no AGENTS/CLAUDE rules).

## Notes

- **Cursor support**: Cursor supports `.claude/commands` and `.claude/skills` (global or project). dotagents also links `.agents/commands` → `.cursor/commands` and `.agents/skills` → `.cursor/skills`.
- **OpenCode preferences**: Uses `~/.config/opencode/AGENTS.md` and prefers AGENTS.md over CLAUDE.md when both exist.
- **Codex prompts**: Always symlink to `.agents/commands` (canonical source).
- **Skills requirements**: Skills require a valid `SKILL.md` with `name` + `description` frontmatter.
- **Claude prompt precedence**: If `.agents/CLAUDE.md` exists, it links to `.claude/CLAUDE.md`. Otherwise `.agents/AGENTS.md` is used. After adding or removing `.agents/CLAUDE.md`, re-run dotagents and apply/repair links to update the symlink. Factory/Codex always link to `.agents/AGENTS.md`.
- **Project scope**: Creates `.agents` plus client folders for commands/hooks/skills only. Rule files (`AGENTS.md`/`CLAUDE.md`) are left to the repo root so you can manage them explicitly.
- **Backups**: Stored under `.agents/backup/<timestamp>` and can be restored via "Undo last change."

## Development

```bash
# Run the CLI in dev mode
bun run dev

# Type-check
bun run type-check

# Run tests
bun test

# Build the CLI
bun run build
```

## Use Cases

- Developers using multiple AI coding assistants who are tired of maintaining duplicate configuration files
- Teams wanting a standardized approach to AI tool configuration
- Projects that need consistent command/skill availability across different AI clients
- Anyone who switches between Claude, Cursor, Factory, Codex, or OpenCode and wants their setup to follow them

## Repository Stats

- **Stars**: 325
- **Language**: TypeScript
- **License**: MIT
- **Full Name**: iannuttall/dotagents
