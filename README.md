# Navigate EVC Platform — Claude Code Skill

A [Claude Code](https://claude.com/claude-code) skill that gives AI agents the ability to autonomously navigate and explore the EV Connect Management Console staging environment using browser automation.

## Quick Start

Give this repo URL to Claude Code and ask it to install the skill:

```
Install the navigate-platform skill from https://github.com/ericzhou129/navigate-evc-platform
```

Claude Code will:
1. Copy the skill into your `.claude/skills/navigate-platform/` directory
2. Prompt you for your staging credentials (email + password)
3. Create a `.env` file with your credentials
4. Set up the OpenBrowser-AI MCP server if not already configured

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

- **[Claude Code](https://claude.com/claude-code)** installed
- **[OpenBrowser-AI](https://github.com/openbrowser-ai/openbrowser)** MCP server (`pip install openbrowser-ai`)
- **Staging credentials** — your `@evconnect.com` account with Management Console access

## What Gets Created

After installation, you'll have:

```
.claude/skills/navigate-platform/SKILL.md   # The skill definition
.env                                         # Your staging credentials (gitignored)
```

The `.env` file contains:
```env
STAGING_URL=https://ops-stage.evconnect.com
STAGING_EMAIL=your-email@evconnect.com
STAGING_PASSWORD=your-password
```

## Usage

The skill activates automatically when Claude Code needs to interact with the staging platform. You can also invoke it directly with `/navigate-platform`.

## Notes

- Zendesk Help Center docs may be outdated — agents verify against what they actually see in the platform
- The React SPA uses unstable element indices — the skill navigates by URL for reliability
- Feature toggles have dependencies: enable at network level first, then organization level
