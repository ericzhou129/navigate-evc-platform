# Navigate EVC Platform — Claude Code Skill

A [Claude Code](https://claude.com/claude-code) skill that gives AI agents the ability to autonomously navigate and explore the EV Connect Management Console staging environment using browser automation.

## What It Does

This skill provides Claude Code agents with complete context for navigating the EV Connect platform:

- **Login flow** — Automated form-based authentication with cookie banner handling
- **Full URL sitemap** — Every page in the Management Console mapped with URL patterns (Platform, Network, Organization, Station levels)
- **Feature catalog** — All 40+ platform plugins/features with categories and dependency rules
- **Station management** — Finding stations, reading details, understanding lifecycle
- **Pricing configuration** — Pricing policy structure, components, restrictions, workflows
- **C-NOC diagnostics** — Station diagnostics metrics and filtering
- **Investigation patterns** — Common question types and step-by-step approaches
- **Zendesk article index** — Key Help Center article IDs for deeper research
- **Troubleshooting guide** — Common issues and fixes for browser automation

## Prerequisites

1. **[OpenBrowser-AI](https://github.com/openbrowser-ai/openbrowser)** MCP server configured in your `.mcp.json`
2. **Staging credentials** in a `.env` file:

```env
STAGING_URL=https://ops-stage.evconnect.com
STAGING_EMAIL=your-agent@evconnect.com
STAGING_PASSWORD=your-password-here
```

3. **Claude Code** installed and configured

## Installation

Copy the skill into your Claude Code skills directory:

```bash
# From your project root
mkdir -p .claude/skills/navigate-platform
cp path/to/SKILL.md .claude/skills/navigate-platform/SKILL.md
```

Or clone this repo and copy:

```bash
git clone https://github.com/ericzhou-evc/navigate-evc-platform.git
cp -r navigate-evc-platform/.claude/skills/navigate-platform .claude/skills/
```

## Usage

The skill activates automatically when Claude Code needs to interact with the staging platform. You can also invoke it directly:

```
/navigate-platform
```

Or reference it from other skills by telling research agents to read `.claude/skills/navigate-platform/SKILL.md` before using OpenBrowser-AI.

## Updating the `.env` Path

The skill references `path/to/your/.env` in the login flow. Update this to match your actual `.env` file location in your project.

## Notes

- Zendesk Help Center documentation may be outdated — the skill instructs agents to verify against what they see in the platform
- Element indices in the React SPA change on every interaction — the skill uses URL-based navigation for reliability
- Some feature toggles have dependencies and must be enabled in a specific order (network level first, then organization level)
