# Story Point Estimation

> Prerequisites: [Skills](guide/03-skills.md) | Time: ~15 min

## What This Does

A skill that estimates the complexity of a task by analyzing the actual codebase, scoring 7 complexity factors, and producing a calibrated Fibonacci story point estimate (1/2/3/5/8/13). No JIRA integration required - just describe the work.

## Setup

### Step 1: Create the skill

```bash
mkdir -p .claude/skills/storypoint
```

Create `.claude/skills/storypoint/SKILL.md`:

```markdown
---
name: storypoint
description: Estimate complexity of a task by analyzing the codebase
model: claude-opus-4-6
---

# /storypoint - Story Point Estimation

**Purpose**: Research a task thoroughly, then estimate its complexity.

## Input
A description of the work to be done (ticket title, feature description, or bug report).

## WORKFLOW

### Step 1: Understand the Task
Parse the description. Identify what needs to change.

### Step 2: Explore the Codebase
Find the affected files, services, and modules. Map the blast radius.

### Step 3: Score Complexity

| Factor | 1 (Low) | 2 (Medium) | 3 (High) |
|--------|---------|------------|----------|
| Files touched | 1-2 | 3-5 | 6+ |
| Services affected | 1 | 2-3 | 4+ |
| External integrations | None | 1 | 2+ |
| Testing complexity | Unit only | + Integration | + E2E |
| Uncertainty | Clear reqs | Some unknowns | Many unknowns |
| Dependencies | None | 1-2 tasks | Blocked chain |
| Risk / Blast radius | Isolated | Shared code | Core system |

**Mapping**: Total 7-9 = 1pt, 10-11 = 2pts, 12-13 = 3pts, 14-16 = 5pts, 17-18 = 8pts, 19-21 = 13pts

### Step 4: Output

## Estimate: [Task Title]
### Recommendation: **X points**

| Factor | Score | Notes |
|--------|-------|-------|
| Files touched | X/3 | [which files] |
| Services affected | X/3 | [which services] |
| External integrations | X/3 | [which APIs] |
| Testing complexity | X/3 | [what tests needed] |
| Uncertainty | X/3 | [what's unclear] |
| Dependencies | X/3 | [what's blocked] |
| Risk / Blast radius | X/3 | [what could break] |
| **Total** | **XX/21** | |

### Affected Files
- [list of files Claude found in the codebase]

### Risks & Unknowns
- [list]

### If > 8 Points
Suggest how to split into smaller tasks.

## RULES
1. Always explore the codebase - don't guess file counts
2. If the task is vague, say so and score Uncertainty as 3
3. If > 8 points, always suggest a split
4. Output only - never modify any files
```

### Step 2: Use it

```
claude
> /storypoint Add pagination to the /users API endpoint
> /storypoint Migrate the auth service from session-based to JWT
> /storypoint Fix the race condition in the background job processor
```

## Try It

Pick a real task from your backlog and run `/storypoint` on it. Compare Claude's estimate with your team's gut feeling. The value isn't in the number being "right" - it's in the structured analysis of blast radius and risks.

## Customize

**Adjust the scoring factors**: If your team cares more about database migrations than external integrations, swap the factors.

**Add calibration**: If you have completed tickets with known story points, add a section to the skill:

```markdown
### Calibration References
After scoring, search the codebase for similar past changes (via git log) and compare.
```

**Batch estimation**: Combine with [Parallel Agents](guide/06-parallel-agents.md) to estimate an entire sprint:

```bash
./scripts/run-parallel.sh \
  "Estimate story points for: {{ITEM}}" \
  "Add pagination to /users" \
  "Fix auth race condition" \
  "Add dark mode toggle"
```
