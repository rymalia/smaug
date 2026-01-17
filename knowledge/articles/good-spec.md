---
title: "How to write a good spec for AI agents"
author: "Addy Osmani"
url: "https://addyosmani.com/blog/good-spec/"
date: "January 13, 2026"
source: "addyosmani.com"
category: "AI Engineering"
tags: ["ai-agents", "specifications", "prompt-engineering", "software-engineering"]
---

# How to Write a Good Spec for AI Agents

**TL;DR:** Aim for a clear spec covering just enough nuance (structure, style, testing, boundaries) to guide AI without overwhelming it. Break large tasks into smaller ones. Plan first in read-only mode, then execute and iterate continuously.

## Core Principle

Developers often struggle with writing AI agent specifications. Simply throwing a massive spec at an AI agent doesn't work—context window limits and the model's "attention budget" get in the way. The key is to write smart specs: documents that guide clearly, stay within practical context sizes, and evolve with the project.

## Five Principles for Great AI Agent Specs

### 1. Start with High-Level Vision, Let AI Draft Details

**Kick off with a concise high-level spec, then have the AI expand it into a detailed plan.**

Instead of over-engineering upfront, begin with a clear goal statement and a few core requirements. Treat this as a "product brief" and let the agent generate a more elaborate spec. This leverages AI's strength in elaboration while you maintain directional control.

**Practical Approach:**
- Start with: "Build a web app where users can track tasks (to-do list), with user accounts, a database, and a simple UI"
- Let the agent respond with: structured overview, feature list, tech stack, data model
- This spec becomes the "source of truth" for both you and the agent

**Use Plan Mode:** Tools like Claude Code offer Plan Mode (read-only) that lets the agent analyze your codebase and create detailed plans without writing code. Start here to draft specs while exploring existing code.

## Key Takeaways

- **Break large tasks into smaller ones** rather than keeping everything in one massive prompt
- **Plan first in read-only mode**, then execute and iterate continuously
- **Treat specs as living documents** that evolve with the project
- **Aim for clarity** covering just enough nuance to guide AI effectively
- **Leverage AI's elaboration ability** while maintaining human directional control

## Source

Referenced by Richard Seroter as "a banger with piles of actionable advice" for anyone working with AI coding agents.
