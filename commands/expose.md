---
description: Expose localhost app via Buddy tunnel
argument-hint: [port]
allowed-tools: Bash(bdy:*,lsof:*), AskUserQuestion
model: sonnet
---

Expose locally running application via Buddy tunnel.

**Arguments:** `$1` - Optional port (auto-detect if not provided)

## Workflow

1. **Create tunnel:** Use `bdy tunnel` command. Follow the **tunnel skill** for details.

2. **Show results:** Public URL

## Quick Reference

```bash
bdy tunnel http localhost:3000              # basic tunnel
bdy tunnel http localhost:3000 -a user:pass # with HTTP basic auth
bdy tunnel http localhost:3000 --buddy      # with Buddy auth
```

> **CLI Tool:** The command is `bdy` (not "buddy"). Always use `bdy tunnel`, `bdy whoami`, etc.