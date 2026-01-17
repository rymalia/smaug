# amp-skills

Collection of Amp AI coding agent skills for marketing, content, and development workflows

**Source:** https://github.com/snarktank/amp-skills

**Stars:** 313

## Description

A collection of specialized skills for Amp, the AI coding agent. Skills are domain-specific instruction sets that extend Amp's capabilities.

## Available Skills

- **ralph** - Autonomous feature development (setup and execution)
- **agent-browser** - Browser automation for testing, form filling, screenshots
- **react-best-practices** - React/Next.js performance optimization (40+ rules from Vercel)
- **web-design-guidelines** - UI/UX audit against 100+ best practices

(React and web design guidelines are symlinked from [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills))

## Usage

Skills are automatically available in Amp. To use one:

```
Use the direct-response-copy skill to write a landing page for my SaaS product.
```

Or trigger naturally:

```
Help me write copy for my landing page.
```

Amp will load the appropriate skill based on your request.

## Adding Skills

Each skill is a folder containing a `SKILL.md` file with frontmatter:

```markdown
---
name: my-skill
description: "Brief description. Use when... Triggers on: keyword1, keyword2."
---

# Skill Title

Instructions, workflows, and patterns go here.
```

## Key Features

- **Ralph Loop** - Autonomous feature development with full setup and execution workflows
- **Agent Browser** - Full browser automation capabilities for testing and form filling
- **Best Practices** - 40+ React/Next.js optimization rules and 100+ UI/UX guidelines
- **Easy Extension** - Simple structure for creating custom skills

## Bookmarked for

Real-world example of Ralph Loop being used in production to ship features with browser-based acceptance testing. Referenced in Ryan Carson's workflow demonstrating /handoff in Amp.

**Bookmarked:** January 15, 2026
**From:** @ryancarson
