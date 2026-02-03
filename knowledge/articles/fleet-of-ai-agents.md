---
title: "I Run a Fleet of AI Agents from a Mac Mini: How I Keep Them From Going Rogue"
type: article
date_added: 2026-02-02
source: https://x.com/i/article/2018394405028364384
author: Rahul Sood
tags: [ai-agents, security, architecture, openai-langchain, openclaw, production-deployment]
via: "Twitter bookmark from @rahulsood"
---

# Running a Fleet of AI Agents Securely on macOS

This article details a production architecture for managing multiple AI agents (one Claude-based Chief of Staff managing two Gemini Flash subordinates) operating autonomously 24/7 on a Mac Mini with security-first design patterns.

## Key Architecture Patterns

### Agent Hierarchy
- **Primary Agent (Claude):** Chief of Staff handling platform development, infrastructure, code audits, and subordinate agent management
- **Subordinate Agents (Gemini Flash):** One for community evangelism, one for general assistance
- All three share the same OpenClaw codebase but run as isolated macOS users with separate ports, configs, and permissions

### Daily Security Audit Pattern
Every morning at 10 AM, a cron job triggers the primary agent to:
1. Pull latest commits from OpenClaw
2. Diff all changed files against previous version
3. Audit for obfuscated code, suspicious network calls, credential changes, postinstall scripts, exfiltration patterns
4. Generate security assessment (SAFE/CAUTION/BLOCK)
5. Report findings to Telegram
6. Only proceed with deployment after human approval

### Workspace Files as Persistent Memory
Each agent maintains files defining identity and behavior:
- **SOUL.md** - Personality, role, boundaries
- **MEMORY.md** - Long-term memory surviving session resets
- **STRATEGY.md** - Domain-specific playbooks
- **HEARTBEAT.md** - Autonomous periodic tasks

The primary agent updates these files for subordinates, enabling zero-reconfiguration when new platform features ship.

## Security Model

### Scoped Permissions
- **Primary Agent:** Limited sudo access (kill, su, launchctl, cp, chown only) for process management
- **Subordinate Agents:** Zero sudo access, confined to their home directories only

### Telegram Access Control
- Primary agent locked to specific group chat ID only (no wildcard additions)
- Messages only processed from allowlist of user IDs
- Only responds to @mentions
- Subordinates: DMs from approved users only, with explicit prompt injection defense

## Implementation Details

The architecture enables:
- Zero-touch security updates with human-in-the-loop approval gates
- Persistent memory surviving session resets and token compactions
- Centralized multi-agent management without SSH
- Defense in depth: OS-level isolation + config-level allowlists + workspace instructions

## Real-World Application

This system powers **boktoshi.com**, an AI trading platform where:
- Agents compete in a live paper trading arena
- Humans can trade alongside the agents
- Security is the foundation, not an afterthought
- Infrastructure management agents also trade and analyze markets

The author notes: *"I can literally spin up a new employee in 5 minutes, and they will fully understand their job like they worked with me for 100 years."*

## Key Takeaways

1. Agent fleets need hierarchical security models, not flat permissions
2. Persistent memory across resets is critical for autonomous systems
3. Human approval gates for automated system updates prevent supply chain attacks
4. Workspace files provide elegant configuration without needing SSH or manual reconfiguration
5. Defense in depth across OS, config, and instruction layers is essential for production deployments

## Links

- [Original X Article](https://x.com/i/article/2018394405028364384)
- [Boktoshi Trading Platform](https://boktoshi.com/)
- [OpenClaw GitHub](https://github.com/google-gemini/gemini-cli)
- [Original Tweet](https://x.com/rahulsood/status/2018394405028364384)
