---
title: "ClawHavoc: 341 Malicious OpenClaw Skills Discovered"
type: article
date_added: 2026-02-02
source: "https://www.koi.ai/blog/clawhavoc-341-malicious-clawedbot-skills-found-by-the-bot-they-were-targeting"
author: "Oren Yomtov (Koi Security)"
tags: [security, malware, openclaw, clawhub, threat-research]
via: "Twitter bookmark from @orenyomtov"
status: critical
---

Koi Security discovered 341 malicious skills on ClawHub targeting OpenClaw bots. The attack campaign, named ClawHavoc, demonstrates sophisticated distribution tactics including prerequisite install chains and platform-specific obfuscation.

## The Attack

335 of the 341 malicious skills originated from ClawHavoc campaign. The attack uses a simple but effective social engineering vector:

1. User sees legit-looking skill (e.g., "clawhub")
2. README instructs to install a "prerequisite" first
3. That prerequisite IS the malware

## Platform-Specific Obfuscation

- **Windows**: Password-protected ZIP files (to evade antivirus scanning)
- **macOS**: Shell scripts that appear official but fetch payload from attacker infrastructure
- The password isn't for user security—it's to hide from automated analysis

## Detection & Prevention

Clawdex is a new skill that validates other skills for malicious signatures before installation. The research team built it to address the gap in ClawHub's security review process.

## Key Findings

- 2,857 total skills audited on ClawHub
- 341 malicious (11.9%)
- 335 from single campaign (ClawHavoc)
- 6 additional isolated malicious skills

## Recommendations

- Always review skill source code before installing
- Use Clawdex to validate skills
- Never install prerequisites without understanding them
- Run skills in sandboxed/isolated environments

## Links

- [Clawdex Security Audit Skill](https://clawdex.koi.security/)
- [Full Research Report](https://www.koi.ai/blog/clawhavoc-341-malicious-clawedbot-skills-found-by-the-bot-they-were-targeting)
- [Original Twitter Thread](https://x.com/orenyomtov/status/2018323558746014087)
