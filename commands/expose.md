---
description: Expose localhost app via Buddy tunnel
argument-hint: [port]
allowed-tools: Bash(bdy:*,lsof:*), AskUserQuestion
model: sonnet
---

Expose locally running application via Buddy tunnel.

**Arguments:** `$1` - Optional port (auto-detect if not provided)

## Workflow

1. **Create tunnel:** Follow the **tunnel skill**

2. **Show results:** Public URL