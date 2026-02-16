# Parallel Agents

> Time: ~15 min | You will learn: **how to run multiple Claude agents in parallel** | You will build: **a shell-based parallel runner**

## What You're Building

A script that runs the same Claude task against multiple items simultaneously. Instead of processing 10 files sequentially (10x the time), all 10 run in parallel (roughly 1x).

## The Concept

Some tasks are "embarrassingly parallel" - the same operation applied to N independent items:

- Review code across 10 files
- Estimate story points for 15 tickets
- Run security audits on 5 services
- Generate documentation for 8 modules

Claude Code supports parallelism in two ways:

**1. Built-in subagents** (inside a session): Claude's Task tool launches parallel child agents within a single conversation. Just ask Claude to do N things at once and it will parallelize automatically.

**2. Shell-based parallelism** (outside a session): Run multiple `claude --print` processes from a shell script. Each process is independent. This is what we'll build.

## Exercise

### Step 1: Understand the pattern

The core idea is simple - background multiple `claude --print` processes:

```bash
echo "Review file A" | claude --print > result-a.md &
echo "Review file B" | claude --print > result-b.md &
echo "Review file C" | claude --print > result-c.md &
wait  # Wait for all background processes to finish
```

Each `&` runs the command in the background. `wait` blocks until all finish.

### Step 2: Build a reusable script

Create `scripts/run-parallel.sh`:

```bash
#!/bin/bash
# Usage: ./scripts/run-parallel.sh <prompt-template> item1 item2 item3 ...
# Template should contain {{ITEM}} as a placeholder.

TEMPLATE=$1
shift

RESULTS_DIR="results/$(date +%Y-%m-%d-%H%M%S)"
mkdir -p "$RESULTS_DIR"

PIDS=()
for ITEM in "$@"; do
    PROMPT="${TEMPLATE//\{\{ITEM\}\}/$ITEM}"
    echo "$PROMPT" | claude --print > "$RESULTS_DIR/$ITEM.md" &
    PIDS+=($!)
    echo "Launched agent for: $ITEM (PID $!)"
done

echo "Waiting for ${#PIDS[@]} agents..."
FAILED=0
for PID in "${PIDS[@]}"; do
    if ! wait "$PID"; then
        ((FAILED++))
    fi
done

echo "Done. $((${#PIDS[@]} - FAILED))/${#PIDS[@]} succeeded."
echo "Results in $RESULTS_DIR/"
```

```bash
chmod +x scripts/run-parallel.sh
```

### Step 3: Try it

Review multiple files in parallel:

```bash
./scripts/run-parallel.sh \
  "Review {{ITEM}} for security issues and code quality" \
  src/api/auth.py src/api/users.py src/api/payments.py
```

Or generate docs for multiple modules:

```bash
./scripts/run-parallel.sh \
  "Write API documentation for the module at {{ITEM}}" \
  src/services/auth.py src/services/billing.py src/services/notifications.py
```

### Step 4: Using built-in subagents (interactive)

Inside a normal Claude session, you can also ask for parallel work:

```
> Review these 5 files for security issues, running each review in parallel:
  - src/api/auth.py
  - src/api/users.py
  - src/api/payments.py
  - src/services/email.py
  - src/utils/crypto.py
```

Claude will automatically launch subagents via the Task tool and collect results.

## Verify It Works

1. Run the parallel script with 3+ items
2. Check that all result files appear in the `results/` directory
3. Verify each result contains a real review (not errors)
4. Compare wall-clock time: 3 parallel reviews should take roughly the same time as 1

## Cost and Rate Limits

Parallel agents multiply your API usage. Keep these in mind:

| Agents | Effect |
|--------|--------|
| 2-5 | Safe for most accounts |
| 5-10 | May hit rate limits on lower-tier plans |
| 10+ | Likely needs rate limit management (add delays between launches) |

To add a delay between launches:

```bash
for ITEM in "$@"; do
    echo "$PROMPT" | claude --print > "$RESULTS_DIR/$ITEM.md" &
    sleep 2  # 2-second delay between launches
done
```

## What You Learned

- **Shell parallelism**: `claude --print &` + `wait` runs multiple agents simultaneously
- **Built-in subagents**: Claude can parallelize automatically inside a session via the Task tool
- Use shell parallelism for batch processing, built-in subagents for interactive work
- Watch API rate limits when running many agents concurrently
- The parallel pattern works with any `claude --print` workflow: reviews, docs, analysis, estimation

---

This completes the tutorial guide. For practical applications, check out the [Recipes](recipes/cloud-debugging.md).
