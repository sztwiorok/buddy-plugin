# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the `skills/` directory of the Buddy Plugin for Claude Code. It contains skills that provide Claude with knowledge about Buddy.works services: Sandboxes (cloud environments) and Tunnels (localhost exposure).

## Skill Structure

Each skill follows this structure:
```
skill-name/
├── SKILL.md              # Main skill file with YAML frontmatter
└── references/           # Supporting reference documentation
    ├── commands.md
    └── examples.md
```

### SKILL.md Format

```yaml
---
name: skill-name
description: Trigger description for when to load this skill
---

# Skill content (markdown)
```

The `description` field determines when Claude automatically loads the skill based on user intent.

## Available Skills

- **bdy-auth**: CLI installation and authentication (`bdy login`, tokens, environment variables)
- **sandbox**: Deploying apps to Buddy Sandbox cloud environments
- **tunnel**: Exposing localhost services via Buddy tunnels

## Critical AI Agent Requirements

### Sandbox Operations
- Always use `--silent` with `bdy sandbox cp` (prevents stdout flood)
- Commands run in background by default; use `--wait` to block until completion
- Apps must bind to `0.0.0.0`, not `127.0.0.1`
- Python on Ubuntu 24.04 requires venv (PEP 668)

### Tunnel Operations
- Always use `run_in_background: true` in Bash tool (tunnels block foreground)
- Ask user about HTTP authentication before creating any HTTP tunnel
- Verify app binds to `0.0.0.0`

### Authentication
- AI agents cannot perform `bdy login` (requires browser interaction)
- User must run `bdy login` in separate terminal, or use token-based auth
- Verify auth with `bdy whoami` before operations

## Key Commands

```bash
# Authentication verification
bdy whoami

# Sandbox lifecycle
bdy sandbox create -i <name> --resources 2x4 --install-command "<cmd>"
bdy sandbox cp --silent ./src <name>:/app
bdy sandbox exec command <name> "<command>"           # runs in background
bdy sandbox exec command <name> "<command>" --wait    # blocks until done
bdy sandbox endpoint add <name> -n web -e 3000
bdy sandbox exec list <name>                          # list commands
bdy sandbox exec logs <name> <cmd-id>                 # view logs

# Tunnel creation
bdy tunnel http localhost:3000 -n <name>
bdy tunnel tcp localhost:5432 -n <name>
```

## Parent Plugin Structure

This skills directory is part of a larger plugin at `../`:
- `../commands/` - Slash commands (`/deploy`, `/expose`)
- `../README.md` - Plugin documentation
- `../.claude/settings.local.json` - Plugin permissions
