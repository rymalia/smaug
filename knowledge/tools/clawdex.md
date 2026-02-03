# Clawdex: OpenClaw Security Scanner

**Category:** Security Tool
**Type:** OpenClaw Defensive Skill
**Link:** https://clawdex.koi.security/
**Testing:** https://glot.io/

## Purpose

Clawdex is a defensive skill for OpenClaw bots that scans and validates skills before installation or use. It helps protect against malicious skills in the ClawHub marketplace.

## Capabilities

### Pre-Installation Scanning
- Check skills before adding to OpenClaw instance
- Detect known malicious skill signatures
- Compare against Indicators of Compromise (IOCs)

### Post-Installation Auditing
- Scan already-installed skills for malicious behavior
- Identify compromised or recently-weaponized skills
- Generate security reports

### Signature Database
- Regularly updated malicious skill database
- Hash-based detection
- IOC feeds integrated
- Community threat intelligence

## How to Use

1. Install Clawdex skill from ClawHub
2. Before installing new skills: `@clawdex scan <skill-name>`
3. Audit existing skills: `@clawdex audit`
4. Review security reports and recommendations

## Known Detections

Clawdex has detected and catalogued:
- 341 malicious skills from ClawHavoc campaign
- Typosquat packages on ClawHub
- Trojanized versions of popular skills
- Backdoored productivity extensions

## Limitations

- Signature-based detection (new variants may evade)
- Requires regular updates to be effective
- Cannot detect all zero-day malicious code
- Behavioral analysis limited to installed skills

## Best Practices

**When Using Clawdex:**
- Scan every new skill before installation
- Review audit reports weekly
- Enable Clawdex on first setup
- Keep skill database updated
- Report suspicious skills to Koi Security team

**Complementary Practices:**
- Run OpenClaw in isolated environments
- Use principle of least privilege
- Monitor bot activity logs
- Disable unused skills
- Use VPS/Docker for production deployment

## Resources

- Interactive testing: https://glot.io/
- Official documentation: https://clawdex.koi.security/
- Threat research: ClawHavoc campaign analysis at Koi Security
