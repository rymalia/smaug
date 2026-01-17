---
title: Home Assistant Manager - Claude Code Skill
source: https://github.com/komal-SkyNET/claude-skill-homeassistant
description: Claude Code skill to supercharge and manage all Home Assistant workflows
stars: 194
language: null
tags:
  - agentic-ai
  - ai-agents
  - automation
  - claude-code
  - home-assistant
---

# Home Assistant Manager - Claude Code Skill

Expert-level Home Assistant configuration management with efficient deployment workflows, remote CLI access, automation verification, and comprehensive Lovelace dashboard development.

**Repository:** https://github.com/komal-SkyNET/claude-skill-homeassistant

## Key Capabilities

### Configuration Management
- Rapid Development Workflow: Deploy changes via `scp` for instant testing, commit to git when stable
- Smart Reload vs Restart: Automatically determines whether to reload or restart based on change type
- Configuration Validation: Always validates before applying changes to prevent downtime
- Remote CLI Access: Seamlessly manages HA instances via SSH and `hass-cli`

### Automation Development
- Complete Verification Protocol: Automatically tests automations by triggering manually and checking logs
- Error Detection: Identifies template errors, type mismatches, and execution failures
- Log Analysis Patterns: Knows what success and error indicators to look for
- Iterative Fix Workflow: Guides through debugging and re-testing cycles

### Lovelace Dashboard Development
- Tablet Optimization: Creates touch-friendly dashboards optimized for specific screen sizes (7", 11", 13")
- Card Type Expertise: Knows when to use Mushroom cards, Tile cards, Panel vs Sections views
- Template Patterns: Provides ready-to-use Jinja2 templates for common use cases
  - Door/window counting with color coding
  - Conditional display based on time/state
  - Multi-condition status indicators
- Common Pitfall Solutions: Solves dashboard registration, auto-entities failures, template type errors
- Real-World Examples: Includes working examples from production tablet dashboards

### Workflow Optimization
- Git + scp Hybrid: Uses git for version control, scp for rapid iteration
- No Restart for Dashboards: Deploys dashboard changes with just browser refresh
- Context7 Integration: Leverages official HA documentation via MCP when available
- Deployment Decision Tree: Guides through the optimal workflow based on change type

## Installation

### Prerequisites
- Claude Code installed and configured
- Home Assistant instance with SSH access enabled
- Git repository connected to `/config` directory
- `hass-cli` installed (`pipx install homeassistant-cli`)
- SSH key authentication configured
- Environment variables: `HASS_SERVER`, `HASS_TOKEN`

### Install the Skill

Clone into your Home Assistant config repository:
```bash
cd /path/to/your/homeassistant/config
mkdir -p .claude/skills
cd .claude/skills
git clone git@github.com:komal-SkyNET/claude-skill-homeassistant.git home-assistant-manager
```

Or download and extract:
```bash
cd /path/to/your/homeassistant/config
mkdir -p .claude/skills/home-assistant-manager
cd .claude/skills/home-assistant-manager
curl -L https://github.com/komal-SkyNET/claude-skill-homeassistant/archive/main.tar.gz | tar xz --strip-components=1
```

## Usage Examples

### Create a New Automation
- Create automation YAML with proper syntax
- Deploy via scp for testing
- Reload automations (no restart needed)
- Manually trigger to test
- Check logs for execution
- Verify notification received
- Commit to git when working

### Build a Tablet Dashboard
- Create new dashboard file in .storage/
- Register in lovelace configuration
- Optimize for target screen size
- Deploy with instant browser refresh
- Test touch interactions
- Iterate on layout and cards

## Integration with Home Assistant

The skill works seamlessly with:
- Home Assistant Core
- YAML configuration files
- Lovelace dashboard system
- Automations engine
- Service calls and templates
- Git-based version control
- SSH remote access

## Workflow Best Practices

1. **Development Cycle**: scp deploy → test → reload → commit when stable
2. **Validation**: Always validate configuration before deployment
3. **Testing**: Manually trigger automations to verify execution
4. **Dashboards**: Use responsive card types for multiple device sizes
5. **Documentation**: Include YAML comments for complex automations

## Related Projects

- [Home Assistant](https://www.home-assistant.io/) - The open-source home automation platform
- [Claude Code](https://github.com/anthropics/claude-code) - AI-powered coding assistant

