# 05: AI Story Point Estimation

## What It Does

Claude deep-researches a JIRA ticket - reading all linked tickets, Confluence docs, and the actual codebase - then produces a calibrated Fibonacci story point estimate (1/2/3/5/8/13).

Also supports batch mode: estimate every ticket in a sprint at once using parallel agents.

## Why It Matters

- Estimates are based on actual codebase analysis, not gut feeling
- Consistent methodology across the team
- Historical calibration against similar completed tickets
- Sprint planning meetings become shorter

## Prerequisites

- JIRA MCP server set up (see [03-jira-confluence-automation](../03-jira-confluence-automation/))

## Setup Steps

### 1. Create the skill

```bash
mkdir -p .claude/skills/storypoint
```

Create `.claude/skills/storypoint/SKILL.md`:

```markdown
---
name: storypoint
description: Deep research a JIRA ticket and provide Fibonacci story point estimate
model: claude-opus-4-6
---

# /storypoint - Story Point Estimation

**Purpose**: Research a ticket thoroughly, then estimate complexity.

## Input
Ticket ID (e.g., `{{ORG}}-123`)

## WORKFLOW

### Step 1: Read Target Ticket
Use jira_get_issue to get full details.

### Step 2: Read ALL Linked Tickets
Read parent epic, child tasks, blocking/blocked-by issues.

### Step 3: Read Referenced Docs
Search Confluence for linked pages. Read any referenced docs.

### Step 4: Search History
Find similar completed tickets:
```
jira_search: "project = {{ORG}} AND status = Done AND summary ~ '<keywords>' ORDER BY resolved DESC"
```
Note their story points for calibration.

### Step 5: Explore Codebase
Identify affected files, services, and blast radius.

### Step 6: Score Complexity

| Factor | 1 (Low) | 2 (Medium) | 3 (High) |
|--------|---------|------------|----------|
| Files touched | 1-2 | 3-5 | 6+ |
| Services affected | 1 | 2-3 | 4+ |
| External integrations | None | 1 | 2+ |
| Testing complexity | Unit only | + Integration | + E2E |
| Uncertainty | Clear reqs | Some unknowns | Many unknowns |
| Dependencies | None | 1-2 tickets | Blocked chain |
| Risk / Blast radius | Isolated | Shared code | Core system |

**Mapping**: 7-9=1pt, 10-11=2pts, 12-13=3pts, 14-16=5pts, 17-18=8pts, 19-21=13pts

### Step 7: Output

## Story Point Estimate: [TICKET-ID]
### Recommendation: **X points**

| Factor | Score | Notes |
|--------|-------|-------|
| [each factor] | X/3 | [detail] |
| **Total** | **XX/21** | |

### Calibration: Similar Completed Tickets
- [TICKET-1] - X points - [summary]

### Risks & Unknowns
- [list]

## RULES
1. Never estimate without reading all linked tickets
2. Always explore codebase - don't guess
3. Always find calibration references
4. If >8 points, suggest splitting
5. Output only - never update JIRA
```

### 2. Use it

```
> /storypoint {{ORG}}-42
```

## See Also

- [explanation.md](explanation.md) - Batch sprint estimation, complexity scoring details
