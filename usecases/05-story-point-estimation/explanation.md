# Explanation: AI Story Point Estimation

## How the Original Enterprise Setup Worked

### Single Ticket: `/storypoint`

The 7-step workflow was strictly enforced:

1. **Read target ticket** - full description, acceptance criteria, labels
2. **Read ALL linked tickets** - parent epics, children, blockers, mentioned tickets (regex `[A-Z]+-\d+`)
3. **Read referenced docs** - Confluence pages, PRDs, architecture docs
4. **Search history** - find 3-5 similar completed tickets via JQL, note their actual points
5. **Explore codebase** - grep for affected classes, count files to modify, check service dependencies
6. **Create implementation plan** - mental outline of what changes are needed
7. **Calculate complexity** - 7 factors, each scored 1-3, total mapped to Fibonacci

The output was a structured markdown report with: recommendation, research summary, implementation plan, complexity breakdown, calibration references, and risks.

### Batch Sprint: `/sprint-estimate`

The sprint estimation skill composed the storypoint skill:

1. **Find sprint** - query JIRA for active/future sprint on the board
2. **Get all tickets** - fetch every ticket in the sprint with current points
3. **Confirm** - if >20 tickets, ask before proceeding (cost/time)
4. **Parallel estimation** - launch one subagent per ticket, all in background
5. **Generate report** - comparison table (AI vs JIRA), variance analysis, distribution

The report highlighted:
- **AI Higher Than JIRA** - tickets that may be underestimated (risk)
- **AI Lower Than JIRA** - tickets that may be overestimated
- **New Estimates** - tickets that had no points at all

### Skills Composition

The sprint-estimate skill called storypoint as a sub-skill:

```python
for ticket in sprint_tickets:
    Task(
        subagent_type="general-purpose",
        prompt=f"/storypoint {ticket.key}",
        run_in_background=True,
        model="opus"
    )
```

This is powerful: you build small, focused skills and compose them into larger workflows.

## The Complexity Matrix

The scoring system works because it converts subjective gut feelings into a structured checklist:

| Factor | Why It Matters |
|--------|---------------|
| **Files touched** | More files = more review, more merge conflicts, more testing |
| **Services affected** | Cross-service changes need coordination and integration testing |
| **External integrations** | Third-party APIs add unpredictability and testing complexity |
| **Testing complexity** | E2E tests take much longer to write and maintain than unit tests |
| **Uncertainty** | Unclear requirements mean more back-and-forth and potential rework |
| **Dependencies** | Blocked tickets create scheduling risk and context switching |
| **Risk / Blast radius** | Changes to shared code affect more teams and need more careful review |

The Fibonacci mapping prevents false precision. A total of 14 and 16 both map to 5 points - the system acknowledges estimation is inherently imprecise.

## Adapting for {{ORG}}

### Without JIRA

If you don't use JIRA yet, the skill still works for ad-hoc estimation:

```
> /storypoint Add rate limiting to the API endpoints

Claude: [explores codebase, identifies affected files]

## Estimate: Add Rate Limiting

### Recommendation: **5 points**

Files affected:
- src/middleware/rate_limit.py (new)
- src/api/routes/*.py (8 files - add decorator)
- tests/test_rate_limit.py (new)
- docker-compose.yml (add Redis)

Complexity: 14/21
- Files: 2/3 (10+ files)
- Services: 2/3 (API + Redis)
- External: 1/3 (Redis is internal)
- Testing: 2/3 (unit + integration)
- Uncertainty: 1/3 (clear requirements)
- Dependencies: 1/3 (none)
- Risk: 2/3 (affects all endpoints)
```

### Calibration

The value of calibration improves over time. After a few sprints, you'll have a library of "{{ORG}}-123 was estimated at 5 and took 3 days" references. Add a calibration section to CLAUDE.md:

```markdown
## Estimation Calibration Reference

| Ticket | Points | Days | Summary |
|--------|--------|------|---------|
| {{ORG}}-42 | 3 | 1.5 | Add user export CSV |
| {{ORG}}-67 | 5 | 3 | Implement rate limiting |
| {{ORG}}-89 | 8 | 5 | Migrate auth to OAuth2 |
```

### Cost Consideration

Each storypoint estimation uses Opus (most expensive model) and can take 2-3 minutes. A batch sprint estimation of 20 tickets could cost $2-5. You can mitigate this with:
- Confirmation before large batches
- Progress reporting during execution
- Saving results to file for offline review
