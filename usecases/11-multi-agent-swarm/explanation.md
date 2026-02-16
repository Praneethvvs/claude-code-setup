# Explanation: Multi-Agent Swarm

## How the Original Enterprise Setup Worked

There are two levels of multi-agent capability:

### Level 1: Built-in Task Tool Parallelism

The `/sprint-estimate` skill used Claude's built-in Task tool to launch parallel subagents:

```python
for ticket in sprint_tickets:
    Task(
        subagent_type="general-purpose",
        prompt=f"/storypoint {ticket.key}",
        run_in_background=True,
        model="opus"
    )
```

Each subagent:
- Gets its own context window
- Can use skills (like `/storypoint`)
- Runs independently in the background
- Returns results to the parent agent

The parent agent tracked progress: "Completed: 15/33 tickets..."

### Level 2: Docker-Based Swarm

A separate Docker-based orchestration system for true parallel execution:

```
claude-swarm/
├── .claude/skills/swarm-test.md    # Test harness
├── scripts/run-swarm.sh            # Orchestrator
└── tests/run-test.sh               # Validation tests
```

A well-designed swarm should have structured test modes:
- **Test 01**: Simple single-agent verification (30s)
- **Test 02**: Agent handoff - passing context between agents (2min)
- **Test 03**: Full multi-agent workflow with build/test (5min)

## Common Swarm Patterns

### Fan-Out / Fan-In

```
Orchestrator → Agent 1 → Result 1 ↘
             → Agent 2 → Result 2 → Aggregator → Final Report
             → Agent 3 → Result 3 ↗
```

Best for: Same operation on N independent items (estimation, review, audit).

### Pipeline

```
Agent A (research) → Agent B (analyze) → Agent C (report)
```

Best for: Multi-stage tasks where each stage has different expertise.

### Map-Reduce

```
Mapper Agents → Chunk Results → Reducer Agent → Summary
```

Best for: Large data processing (analyze 100 files, summarize results).

### Supervisor

```
Supervisor → Worker 1
           → Worker 2
           → Worker 3
[Supervisor monitors, redistributes failed tasks]
```

Best for: Unreliable tasks that may need retry.

## Skills as Contracts

Skills are critical for swarms because they provide **consistency across agents**. When 30 agents all run `/storypoint`:
- They follow the same workflow
- They use the same scoring matrix
- They produce identically structured output
- They follow the same safety rules

Without skills, each agent might approach the task differently, making results incomparable.

## Cost Management

Each agent in a swarm consumes tokens independently. Cost scales linearly with agent count.

| Model | ~Cost per Agent (simple task) | 20-ticket sprint estimate |
|-------|-------------------------------|--------------------------|
| Haiku | ~$0.01 | ~$0.20 |
| Sonnet | ~$0.05 | ~$1.00 |
| Opus | ~$0.15 | ~$3.00 |

Mitigations:
- **Model selection**: Use Opus only for complex tasks, Haiku for simple ones
- **Confirm before launch**: "Found 33 tickets. This will cost ~$5. Proceed?"
- **Progress reporting**: Let users cancel mid-swarm if results look wrong
- **Caching**: Save results to files so re-runs don't re-process completed items

## When to Use Swarms vs Single Agent

| Scenario | Recommendation |
|----------|---------------|
| Task takes < 5 minutes | Single agent |
| Same operation on N independent items | Swarm (fan-out/fan-in) |
| Deep context about one complex thing | Single agent |
| Breadth across many independent things | Swarm |
| Budget is tight | Single agent |
| Time is critical | Swarm |

## Adapting for {{ORG}}

### Start with Task Tool

Don't build Docker swarms on day one. Use the built-in Task tool:

```
You: Review all Python files in src/api/ for security issues

Claude: [launches parallel agents for each file]
```

### Graduate to Shell Scripts

When you need scheduled or CI-triggered swarms:

```bash
#!/bin/bash
# Parallel security audit across services
SERVICES=("api" "worker" "data-processor")
for svc in "${SERVICES[@]}"; do
    echo "Audit src/$svc/ for OWASP Top 10 vulnerabilities" | \
        claude --print > "reports/audit-$svc.md" &
done
wait
echo "All audits complete"
```

### Graduate to Docker

When you need isolation and reproducibility for production automation.
