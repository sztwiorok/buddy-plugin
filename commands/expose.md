---
description: Expose localhost app via Buddy tunnel
argument-hint: [port]
allowed-tools: Bash(bdy:*,lsof:*), AskUserQuestion
model: sonnet
---

Expose locally running application via Buddy tunnel.

**Arguments:** `$1` - Optional port (auto-detect if not provided)

## CRITICAL: AI Agent Requirements

### 1. Run tunnel in background (MANDATORY)

Tunnel commands BLOCK execution. You MUST use `run_in_background: true`:

```
Bash tool call:
  command: bdy tunnel http localhost:<port>
  run_in_background: true
```

### 2. Ask about authentication (MANDATORY)

Before creating tunnel, use AskUserQuestion to ask about authentication:

- **"HTTP Basic Auth"** → use `-a user:pass` flag
- **"Buddy Auth"** → use `--buddy` flag
- **"No auth (public)"** → no auth flag

## Workflow

1. **Detect port:** Auto-detect with `lsof` or use provided `$1`
2. **Ask about auth:** Use AskUserQuestion (see above)
3. **Create tunnel:** `bdy tunnel http localhost:<port> [auth-flags]` with `run_in_background: true`
4. **Show results:** Public URL

## Quick Reference

```bash
bdy tunnel http localhost:3000              # basic tunnel
bdy tunnel http localhost:3000 -a user:pass # with HTTP basic auth
bdy tunnel http localhost:3000 --buddy      # with Buddy auth
```

> **CLI Tool:** The command is `bdy` (not "buddy"). Always use `bdy tunnel`, `bdy whoami`, etc.
