---
name: sandbox-agent
description: Delegate AI tasks to Claude agents running in isolated Buddy Sandbox environments. Use when user asks to "delegate task", "run task in sandbox", "parallel agent execution", "isolated AI task", "YOLO mode", "sub-agent", "compare solutions", "multi-agent", or mentions running AI tasks in sandboxes.
---

# Sandbox Agent - Task Delegation

Delegate AI tasks to Claude agents running in isolated Buddy Sandbox environments. Enable YOLO-mode execution, parallel processing, and multi-agent comparison.

## Benefits

- **YOLO Mode**: Sub-agent executes any command without permission prompts (isolated environment)
- **Isolation**: Mistakes don't affect your main environment
- **Parallelism**: Multiple agents work simultaneously on different sandboxes
- **Multi-agent Comparison**: Run same task on multiple agents, compare results, pick best or merge
- **Read-only Main Agent**: Only reads results from logs/files, clean separation

## CRITICAL: Mandatory User Questions

> **STOP: Before delegating ANY task, you MUST ask the user these questions using AskUserQuestion tool.**

### Question 1: Sandbox Source

"How should I create the sandbox for this task?"

Options:
1. **Fresh sandbox (Ubuntu 24.04)** - Clean environment, start from scratch
2. **From snapshot** - Use existing snapshot (user specifies name)
3. **Use existing sandbox** - Reuse running sandbox (user specifies ID)

### Question 2: Pre-task Setup

"Any setup commands to run before the main task?"

Options:
1. **None** - Proceed directly to task
2. **Custom setup** - User specifies commands (e.g., "Install Node.js 20", "Clone repo X")

### Question 3: Parallel Execution

"Run this task on multiple sandboxes?"

Options:
1. **Single sandbox** - One agent execution
2. **Multiple sandboxes** - Parallel execution for comparison (user specifies count and focus)

**DO NOT skip these questions. DO NOT proceed until user has made choices.**

## Core Commands

### Create Sandbox

```bash
# Fresh sandbox
bdy sandbox create -i <name> --resources 4x8 --wait-for-configured

# From snapshot
bdy sandbox create -i <name> --snapshot <snapshot-name> --resources 4x8 --wait-for-configured

# With install commands
bdy sandbox create -i <name> --resources 4x8 \
  --install-command "apt-get update && apt-get install -y nodejs npm" \
  --wait-for-configured
```

Resources: `2x4` (light), `4x8` (standard), `8x16` (heavy). Format: CPUxRAM.

### Delegate Task to Claude

```bash
# Standard execution
bdy sandbox exec command <sandbox-id> "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"YOUR TASK HERE\""

# With streaming output for live monitoring
bdy sandbox exec command <sandbox-id> "sudo -u claude -i -- claude --dangerously-skip-permissions --output-format stream-json -p \"YOUR TASK HERE\""
```

**Important flags:**
- `sudo -u claude -i` - Switch to claude user with login shell
- `--dangerously-skip-permissions` - Enable YOLO mode (safe in isolated sandbox)
- `-p "..."` - Pass task prompt

### Continue Session (Follow-up)

```bash
bdy sandbox exec command <sandbox-id> "sudo -u claude -i -- claude -c"
```

The `-c` flag continues the most recent session, maintaining full context.

### Monitor Progress

```bash
# List all commands (get command IDs)
bdy sandbox exec list <sandbox-id>

# Check status (PENDING/INPROGRESS/SUCCESS/FAILED)
bdy sandbox exec status <sandbox-id> <command-id>

# Read current output (can poll repeatedly for live progress)
bdy sandbox exec logs <sandbox-id> <command-id>

# Wait for completion and get full output
bdy sandbox exec logs <sandbox-id> <command-id> --wait
```

### Read Results

```bash
# View file created by sub-agent
bdy sandbox exec command <sandbox-id> "cat /path/to/result" --wait

# List directory
bdy sandbox exec command <sandbox-id> "ls -la /path/to/dir" --wait

# Copy results back to local machine
bdy sandbox cp <sandbox-id>:/remote/path ./local/path --silent
```

### Cleanup

```bash
# Kill running task
bdy sandbox exec kill <sandbox-id> <command-id>

# Destroy sandbox
bdy sandbox destroy <sandbox-id>
```

## Live Monitoring Pattern

Main agent can monitor progress by polling logs:

```bash
# Start task
bdy sandbox exec command my-sandbox "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Complex task...\""
# Returns command ID

# Poll for progress (repeat as needed)
bdy sandbox exec status my-sandbox <cmd-id>    # Check if still INPROGRESS
bdy sandbox exec logs my-sandbox <cmd-id>      # Read current output

# When status is SUCCESS/FAILED, get final output
bdy sandbox exec logs my-sandbox <cmd-id> --wait
```

## Multi-Agent Pattern

For comparing solutions or getting multiple perspectives:

```bash
# 1. Create multiple sandboxes in parallel
bdy sandbox create -i agent-1 --resources 4x8 --wait-for-configured &
bdy sandbox create -i agent-2 --resources 4x8 --wait-for-configured &
bdy sandbox create -i agent-3 --resources 4x8 --wait-for-configured &
wait

# 2. Delegate tasks (same or with different focus)
bdy sandbox exec command agent-1 "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Task with focus A\""
bdy sandbox exec command agent-2 "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Task with focus B\""
bdy sandbox exec command agent-3 "sudo -u claude -i -- claude --dangerously-skip-permissions -p \"Task with focus C\""

# 3. Wait for all and collect results
bdy sandbox exec logs agent-1 <cmd-id> --wait
bdy sandbox exec logs agent-2 <cmd-id> --wait
bdy sandbox exec logs agent-3 <cmd-id> --wait

# 4. Main agent compares/aggregates results

# 5. Cleanup
bdy sandbox destroy agent-1 && bdy sandbox destroy agent-2 && bdy sandbox destroy agent-3
```

## Prerequisites

**Authentication Required:** Verify with `bdy whoami`. If fails, user must run `bdy login` in separate terminal.

## Advanced Features

For advanced sandbox features, see the **sandbox skill**:
- Endpoint exposure (`bdy sandbox endpoint add`)
- Snapshot management (`bdy sandbox snapshot create/list`)
- Detailed file transfer options
- Resource sizing recommendations

## References

- [references/examples.md](references/examples.md) - Complete examples (simple task, test generation, multi-agent review)
