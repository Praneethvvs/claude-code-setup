# Explanation: Self-Improving Configuration

## How the Original Enterprise Setup Worked

A well-designed `/reflect` skill can be more sophisticated than a simple review. It solves a critical problem: **context loss in long sessions**.

### The Context Recovery Problem

During long Claude Code sessions, earlier messages get "compacted" (compressed) to fit the context window. By the end of a 2-hour session, Claude may not remember what happened in the first 30 minutes.

The original skill worked around this by reading the raw session transcript:

```bash
# Find the session file
ls -lt ~/.claude/projects/*/*.jsonl | grep -v agent | head -5

# Extract all user messages from the JSONL transcript
grep '"role":"user"' /path/to/session.jsonl | head -100

# Find compaction summaries (compressed earlier context)
grep '"Summary:"' /path/to/session.jsonl
```

This gave the reflect skill access to the FULL conversation, not just what was still in context. It could identify patterns across the entire session.

**Caveat**: This approach reads Claude Code internal files (JSONL transcripts) whose format may change between versions. The simplified approach (reviewing current context only) is more stable and works well enough for most teams. Only use transcript reading if you need to recover information from very long sessions.

### What It Looked For

| Category | Example |
|----------|---------|
| **New patterns** | "We discovered that CloudWatch queries timeout if you don't limit the time range" |
| **Recurring mistakes** | "Had to fix the same import error 3 times" |
| **Missing docs** | "Spent 20 min finding the right database connection string format" |
| **New tools** | "Used `az monitor log-analytics` which isn't documented in our guide yet" |
| **Workflow improvements** | "Running tests with `--filter` was much faster than full suite" |
| **Clarifications** | "The 'staging' environment actually shares some prod secrets" |

### Output and Storage

Suggestions were saved to `wip/claude-md-suggestions/{date}-{topic}.md`, not applied directly. This ensures:
- Human review before changes
- No accidental CLAUDE.md corruption
- Suggestions can be discussed with the team
- A history of what was learned

### Tool Restrictions

```yaml
allowed-tools: Read, Glob, Grep, Bash
```

The skill was restricted to read-only tools. It could explore the codebase and read files, but couldn't edit anything. This is a safety measure - reflection should suggest, not act.

## Claude Features That Enable This

### Skills with Tool Restrictions

The `allowed-tools` field prevents the reflect skill from accidentally modifying files while analyzing the session. This is a good pattern for any skill that should observe but not modify.

### Model Selection (Opus)

Retrospective analysis requires:
- Reading and synthesizing long conversation histories
- Identifying subtle patterns across many interactions
- Writing clear, actionable improvement suggestions

Opus is the right choice here - thoroughness matters more than speed.

### Session Transcript Access

Claude Code stores session transcripts as JSONL files in `~/.claude/projects/`. These are the raw conversation including tool calls, results, and compaction summaries. Reading these gives the reflect skill perfect recall.

## Adapting for {{ORG}}

### Simplified Version (No Transcript Reading)

If you don't need full transcript recovery, the simplified skill in the README works well. It reviews what's currently in context and suggests improvements.

### Scheduled Reflection

Consider reflecting at specific milestones:
- End of each day
- After resolving a complex bug
- After onboarding a new service
- After a sprint retrospective

### Team-Wide Improvements

If multiple team members use Claude Code, collect reflect suggestions in a shared location:

```
docs/claude-improvements/
├── 2026-02-15-auth-debugging.md      # Praneeth's session
├── 2026-02-16-api-migration.md       # Another teammate
├── 2026-02-17-sprint-planning.md     # Another session
```

Review these during sprint retros and apply the best ones to the shared CLAUDE.md.

### What Makes a Good CLAUDE.md Improvement

**Good**: Specific, actionable, prevents repeat issues
```
Add to ## Common Issues:
"When `alembic upgrade head` fails with 'multiple heads',
run `alembic merge heads` to create a merge migration."
```

**Bad**: Vague, obvious, or too specific to one session
```
"Add note about being careful with database migrations"
```
