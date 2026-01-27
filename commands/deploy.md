---
description: Deploy app to Buddy Sandbox from current directory
argument-hint: [sandbox-name] [path-to-deploy]
allowed-tools: Bash(bdy:*), Read, Glob, AskUserQuestion
model: sonnet
---

Deploy application to Buddy Sandbox.

**Arguments:**
- `$1`: Optional sandbox name
- `$2`: Optional path to deploy (default: current directory)

## Workflow

1. **Analyze project:** Understand what to deploy, dependencies, start command, port

2. **Deploy:** Use `bdy sandbox` commands. Follow the **sandbox skill** for details.

3. **Show results:** Sandbox ID, public URL, how to view logs

## Quick Reference

```bash
bdy sandbox create -i my-app --resources 2x4 --wait-for-running
bdy sandbox cp ./src my-app:/app
bdy sandbox exec command my-app "npm install && npm start"
```

> **CLI Tool:** The command is `bdy` (not "buddy"). Always use `bdy sandbox`, `bdy whoami`, etc.
