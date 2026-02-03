# OpenClaw: Autonomous AI Agent Framework

**Category:** AI Agent Platform
**Links:**
- Clawdex Security Tool: https://clawdex.koi.security/
- Koi Security Blog: https://www.koi.ai/blog/clawhavoc-341-malicious-clawedbot-skills-found-by-the-bot-they-were-targeting

## Overview

OpenClaw is a framework for building autonomous AI agents that run 24/7 on local machines. The ecosystem includes:
- Skill marketplace (ClawHub)
- Pre-built agent templates
- Integration with messaging platforms (WhatsApp, Telegram)
- Access to local resources (files, calendars, email)

## Key Capabilities

### What OpenClaw Agents Can Do

**Local Machine Operations:**
- File system access and manipulation
- Calendar and email integration
- Notes and knowledge base interaction
- System command execution

**Integration Capabilities:**
- 24/7 operation on user's machine
- Reachable via WhatsApp/Telegram
- Proactive task execution
- Background monitoring

### OpenClaw Executive Assistant System Prompt

An effective OpenClaw implementation includes:
- Cost awareness and token economy rules
- Security boundaries for safe operation
- Proactive behavior configuration
- Communication style guidelines (no filler, focus on outcomes)
- Approval workflows for expensive operations

## Ecosystem

### ClawHub Skills Marketplace
- Community-contributed extensions
- Permissive publishing model
- Subject to supply chain attacks (see ClawHavoc research)

### Security Considerations

**Risks:**
- Direct access to sensitive local data
- Ability to execute system commands
- Vulnerability to malicious skills
- Supply chain attack surface

**Mitigations:**
- Use Clawdex to scan skills before installation
- Run in isolated environments (Docker/VPS, not main machine)
- Review skill permissions before enabling
- Monitor bot activity and access logs

## Production Deployment

**Recommended Setup:**
- Run in Docker container
- Deploy on isolated VPS
- Regular security audits
- Skill scanning before updates

**Cost Optimization:**
- Estimated 40% token cost reduction with well-designed prompts
- Batch operations to reduce API calls
- Cache frequently-accessed data
- Use local operations over API calls when possible

## Resources

- Security tool: Clawdex (https://clawdex.koi.security/)
- Live testing: https://glot.io/
- Security research: ClawHavoc threat analysis
