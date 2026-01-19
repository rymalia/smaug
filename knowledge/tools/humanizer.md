---
title: Humanizer
url: https://github.com/blader/humanizer
author: blader (Siqi Chen)
stars: 405
language: N/A
topics: []
archived: Sunday, January 18, 2026
tweet: https://x.com/blader/status/2013015738622284156
---

# Humanizer

**Claude Code skill that removes signs of AI-generated writing from text**

A Claude Code skill that detects and eliminates 24 patterns of AI-generated writing identified by Wikipedia's WikiProject AI Cleanup. Based on observations of thousands of AI-generated text instances, it transforms statistically-likely AI output into natural human writing.

## Key Features

### 24 Detected Patterns

**Content Patterns:**
- Significance inflation ("pivotal moment in evolution" → "established in 1989")
- Notability name-dropping (vague citations → specific sourced quotes)
- Superficial -ing analyses (remove "symbolizing, reflecting, showcasing")
- Promotional language ("breathtaking region" → "town in Gonder region")
- Vague attributions ("Experts believe" → "according to 2019 survey by...")
- Formulaic challenges (generic obstacles → specific facts)

**Language Patterns:**
- AI vocabulary (replace "testament, landscape, showcasing")
- Copula avoidance ("serves as, boasts" → "is, has")
- Negative parallelisms ("not just X, it's Y" → direct statement)
- Rule of three (arbitrary triplets → natural counts)
- Synonym cycling (excessive variation → clear repetition)
- False ranges (exaggerated scope → direct listing)

**Style Patterns:**
- Em dash overuse (convert to commas/periods)
- Boldface overuse (remove excessive emphasis)
- Inline-header lists (convert to prose)
- Title Case Headings (use sentence case)
- Emojis (remove from formal text)
- Curly quotes (convert to straight quotes)

**Communication Patterns:**
- Chatbot artifacts ("I hope this helps!" → remove)
- Cutoff disclaimers ("limited sources" → find or remove)
- Sycophantic tone ("Great question!" → direct response)

**Filler and Hedging:**
- Filler phrases ("in order to" → "to")
- Excessive hedging ("could potentially possibly" → "may")
- Generic conclusions ("future looks bright" → specific facts)

## Installation

```bash
# Clone directly into Claude Code skills directory
mkdir -p ~/.claude/skills
git clone https://github.com/blader/humanizer.git ~/.claude/skills/humanizer
```

## Usage

```
/humanizer

[paste your text here]
```

Or: "Please humanize this text: [your text]"

## Core Insight

From Wikipedia's guide:
> "LLMs use statistical algorithms to guess what should come next. The result tends toward the most statistically likely result that applies to the widest variety of cases."

This explains why AI writing sounds generic - it optimizes for broad applicability rather than specific accuracy.

## Example Transformation

**Before:**
> The new software update serves as a testament to the company's commitment to innovation. Moreover, it provides a seamless, intuitive, and powerful user experience—ensuring that users can accomplish their goals efficiently. It's not just an update, it's a revolution in how we think about productivity.

**After:**
> The software update adds batch processing, keyboard shortcuts, and offline mode. Early feedback from beta testers has been positive, with most reporting faster task completion.

## Why This Matters

Wikipedia editors have curated a detailed playbook of AI writing detection based on real-world observation. By feeding this back to LLMs with explicit instructions to avoid these patterns, we get output that sounds naturally human rather than statistically average.

## References

- [Wikipedia: Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) - Primary source maintained by WikiProject AI Cleanup
