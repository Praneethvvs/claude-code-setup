# 11: Multi-Agent Swarm

## What It Does

Runs multiple Claude agents in parallel to tackle large tasks that are too big for one agent. Each agent handles a piece, results are collected and combined.

Examples:
- Estimate all 20 tickets in a sprint simultaneously
- Review code across 10 repositories at once
- Run security audits on multiple services in parallel

## Why It Matters

Some tasks are "embarrassingly parallel" - the same operation on N independent items. Running them sequentially takes N times as long. Parallel agents cut that to roughly 1x.

## Prerequisites

- Claude Code installed
- Understanding of the Task tool (subagents)

## How It Works

### Option 1: Built-in Subagents (Simplest)

Claude's Task tool launches parallel subagents within a single session:

```
You: Estimate story points for {{ORG}}-10, {{ORG}}-11, {{ORG}}-12, {{ORG}}-13, {{ORG}}-14

Claude: Launching 5 estimation agents in parallel...
[Each agent runs /storypoint independently]
[Results collected as they complete]

Completed 5/5:
| Ticket | Points | Key Risk |
|--------|--------|----------|
| {{ORG}}-10 | 3 | Clear requirements |
| {{ORG}}-11 | 5 | External API dependency |
| {{ORG}}-12 | 2 | Simple UI change |
| {{ORG}}-13 | 8 | Core auth system change |
| {{ORG}}-14 | 3 | Database migration needed |

Total: 21 points
```

### Option 2: Shell-Based Parallelism

Run multiple Claude instances from the command line:

```bash
#!/bin/bash
# run-swarm.sh - Run Claude on multiple items in parallel

ITEMS=("{{ORG}}-10" "{{ORG}}-11" "{{ORG}}-12" "{{ORG}}-13" "{{ORG}}-14")
RESULTS_DIR="results/$(date +%Y-%m-%d)"
mkdir -p "$RESULTS_DIR"

for item in "${ITEMS[@]}"; do
    echo "Estimate story points for $item" | \
        claude --print > "$RESULTS_DIR/$item.md" &
done

wait
echo "All done. Results in $RESULTS_DIR/"
```

### Option 3: Docker Swarm (Full Isolation)

```bash
for item in "${ITEMS[@]}"; do
    docker run -d --name "agent-$item" \
        -v $(pwd):/workspace \
        {{org}}-claude-agent \
        "Estimate story points for $item" &
done
wait
for item in "${ITEMS[@]}"; do
    docker logs "agent-$item" > "results/$item.md"
done
```

## Setup Steps

### 1. Define the unit of work

Create a skill or prompt for the single-item operation (e.g., `/storypoint`).

### 2. Create the orchestrator

For built-in subagents, create a skill that launches parallel Task agents:

```markdown
---
name: batch-estimate
description: Estimate all tickets in a sprint in parallel
---

# /batch-estimate

## WORKFLOW

### Step 1: Get Sprint Tickets
Use jira_get_sprint_issues to find all tickets.

### Step 2: Confirm
"Found N tickets. Launching N parallel agents. Proceed?"

### Step 3: Launch Parallel Agents
For each ticket, launch a background Task agent running /storypoint.

### Step 4: Collect Results
Wait for all agents. Report progress as they complete.

### Step 5: Generate Summary
Combine all results into a comparison table.
```

### 3. Use it

```
> /batch-estimate current sprint
```

## See Also

- [explanation.md](explanation.md) - Patterns, cost management, when to use swarms vs single agents
