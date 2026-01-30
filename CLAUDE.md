# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** that integrates with the Buddy.works cloud platform. It enables deploying applications to cloud sandboxes, exposing localhost services via tunnels, and delegating AI tasks to isolated agents.

**Architecture:** Skills-first design where skills contain domain knowledge and workflows, commands are lightweight user entrypoints.

## Key Commands

All operations use the `bdy` CLI (not "buddy"):

```bash
# Authentication (must run in separate terminal - interactive browser auth)
bdy login
bdy whoami

# Sandbox management
bdy sandbox create <name> --wait-for-running
bdy sandbox cp <source> <sandbox>:<dest> > /dev/null 2>&1
bdy sandbox exec <sandbox> -- <command>
bdy sandbox rm <name> --force

# Tunnel management
bdy tunnel http <port>
bdy tunnel tcp <port>
bdy tunnel tls <port>
```

## Critical Implementation Patterns

### Authentication
- Always verify `bdy whoami` before operations
- Users must run `bdy login` in a **separate terminal** (AI cannot do interactive OAuth)

### Sandbox Deployment
- Use `--wait-for-running` or `--wait-for-configured` when creating sandboxes
- Redirect copy output: `bdy sandbox cp ... > /dev/null 2>&1` (prevents stdout flooding)
- Applications must bind to `0.0.0.0` (not `127.0.0.1`) for endpoints to work

### Tunnel Execution (CRITICAL)
- **ALWAYS** use `run_in_background: true` parameter when running tunnel commands
- Tunnel commands block execution if run in foreground
- Ask about authentication (HTTP Basic, Buddy, or None) via AskUserQuestion before creating

### Multi-Agent Delegation
- Create sandboxes in parallel with `&` and `wait`
- Delegate with: `bdy sandbox exec <name> -- sudo -u claude -i -- claude ...`
- Use `--dangerously-skip-permissions` (safe in isolated sandbox environments)

## Project Structure

```
buddy-plugin/
├── commands/           # User commands (/deploy, /expose, /delegate)
├── skills/
│   ├── sandbox/        # Deployment skill with tech-stack examples
│   ├── tunnel/         # Tunnel skill with auth options
│   └── sandbox-agent/  # Multi-agent delegation patterns
├── .claude/            # Plugin permissions
└── .claude-plugin/     # Marketplace metadata
```

## Development

**Testing during development:**
```bash
cc --plugin-dir /path/to/buddy-plugin
```

**Permanent installation:**
```bash
git clone https://github.com/sztwiorok/buddy-plugin.git ~/.claude/plugins/buddy
```

No build step - changes to .md files take effect on next Claude Code session.

## Permission Configuration

`.claude/settings.local.json` controls allowed Bash commands. Key design:
- Main plugin allows wide range of bdy commands
- Prevents agents from calling "buddy" (must use "bdy")
- Skills subdirectory has narrower permissions for safety isolation
