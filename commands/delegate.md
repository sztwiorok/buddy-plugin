---
description: Delegate AI task to Claude agent in sandbox
argument-hint: <task-description>
allowed-tools: Bash(bdy:*), Read, Glob, AskUserQuestion
model: sonnet
---

Delegate an AI task to a Claude agent running in an isolated Buddy Sandbox environment.

**Arguments:**
- `$1`: Task description to delegate (optional - will ask if not provided)

## Workflow

1. **Ask user questions** per the **sandbox-agent skill** requirements:
   - Sandbox source (fresh / snapshot / existing)
   - Pre-task setup commands
   - Single or multiple sandboxes (for comparison)

2. **Setup sandbox(es):**
   - Create new sandbox(es) or use existing
   - Run any setup commands specified by user
   - Wait for configuration to complete

3. **Delegate task:**
   - Execute Claude in sandbox with task prompt
   - Use `sudo -u claude -i -- claude --dangerously-skip-permissions -p "..."`

4. **Monitor and report:**
   - Show command ID for tracking
   - Provide commands to check progress
   - Wait for completion if requested

5. **Return results:**
   - Show task output/logs
   - For multi-agent: aggregate and compare results
   - Provide cleanup commands

## Example Usage

```
/delegate "Generate unit tests for my API"
/delegate "Review this code for security issues"
/delegate "Refactor to use async/await"
```

## Reference

Follow the **sandbox-agent skill** for detailed command reference and patterns.
