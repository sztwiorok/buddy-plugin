---
description: Deploy app to Buddy Sandbox from current directory
argument-hint: [sandbox-name] [path-to-deploy]
allowed-tools: Bash(bdy:*), Read, Glob, AskUserQuestion
model: sonnet
---

Deploy application to Buddy Sandbox. Use the **sandbox skill** for deployment procedures and best practices.

**Arguments:**
- `$1`: Optional sandbox name (default: generate from directory/timestamp)
- `$2`: Optional path to deploy (default: current directory)

## Workflow

### 1. Verify Authentication

Check authentication with:
```bash
bdy workspace ls
```

If fails, inform user: "Run `bdy login` in a separate terminal (AI agents cannot perform interactive login). See **bdy-auth skill** for details."

### 2. Analyze Project

Examine the directory to understand:
- What application(s) need deployment
- Required dependencies and runtime
- How to start the application
- What port(s) to expose

If user specified a subdirectory in `$2`, deploy from that path. Otherwise use current directory.

**Note:** Repository may contain multiple deployable applications (e.g., blog + admin panel). Ask user which to deploy if unclear.

### 3. Deploy Using Sandbox Skill

Use the **sandbox skill** procedures to:
- Create sandbox with appropriate resources
- Copy application files with `bdy sandbox cp --silent`
- Install dependencies
- Start application in detached mode
- Expose HTTP endpoint

The sandbox skill contains complete procedures for deployment.

### 4. Display Results

Show:
- Sandbox identifier
- Public URL
- How to view logs: `bdy sandbox command logs [id] --last`
- Management commands (stop, destroy)

## Important Notes

- Always use `--silent` with `bdy sandbox cp`
- Applications must bind to `0.0.0.0`, not `127.0.0.1`
- Ubuntu 24.04 requires venv for Python pip packages
- Endpoints served via HTTPS proxy

## Error Handling

If deployment fails:
- Show error from bdy CLI
- Check logs with `bdy sandbox command logs [id] --last`
- For auth errors: direct user to run `bdy login` in separate terminal
- For name conflicts: suggest unique name or destroy existing sandbox
