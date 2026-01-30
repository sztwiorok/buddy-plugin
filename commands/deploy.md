---
description: Deploy app to Buddy cloud
argument-hint: [name] [path]
allowed-tools: Bash(bdy:*), Read, Glob, AskUserQuestion
model: sonnet
---

Deploy application to Buddy cloud.

**Arguments:**
- `$1`: Optional name (sandbox or package name)
- `$2`: Optional path to deploy (default: current directory)

## Workflow

1. **Analyze project:** Understand what to deploy - dependencies, build command, output directory, whether it needs a server

2. **Ask deployment type:** Use AskUserQuestion to determine deployment method:
   - **"Static site (HTML/CSS/JS files)"** → use **static-site skill** (Buddy Packages)
   - **"Dynamic application (needs server)"** → use **sandbox skill** (Buddy Sandbox)

3. **Deploy:** Follow the appropriate skill based on user choice

4. **Show results:** URL, how to update/manage the deployment

## When to Use Which

| Type | Use Case | Skill |
|------|----------|-------|
| Static | Pre-built HTML/CSS/JS, SPAs, static exports | static-site |
| Dynamic | Node.js servers, Python apps, databases | sandbox |

## Quick Reference

**Static site (package):**
```bash
npm run build
bdy package publish my-site@1.0.0 ./dist --create
```

**Dynamic app (sandbox):**
```bash
bdy sandbox create -i my-app --resources 2x4 --wait-for-running
bdy sandbox cp ./src my-app:/app > /dev/null 2>&1
bdy sandbox exec command my-app "npm install && npm start"
```

> **CLI Tool:** The command is `bdy` (not "buddy"). Always use `bdy sandbox`, `bdy package`, `bdy whoami`, etc.
