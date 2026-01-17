# Paul Solt's Multi-Agent Collaboration Patterns

**Source:** https://gist.github.com/PaulSolt/ca41c147a2c65856bfb39d9705e1de24

## Overview

Patterns and configurations for working with multiple Codex agents in the same project folder, including task coordination, atomic commits, and agent rules.

## Key Files

- `Tasks.md` - Task definitions and assignment
- `atomic_commit.sh` - Script for atomic git commits
- `Agents.md` - Agent configuration and cooperation rules

## Core Principles

1. Keep things civil with explicit rules for agent coordination
2. One active agent at a time (sequential primary tasks)
3. Second agent can handle research or documentation in parallel
4. Shared context through markdown documentation
5. Atomic commits for clear change tracking

## Use Cases

- Managing multiple AI agents on feature development
- Coordinating between coding and documentation tasks
- Preventing agent conflicts through task sequencing
- Clean git history with atomic commits
