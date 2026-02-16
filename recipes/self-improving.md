# Self-Improving Config

> Prerequisites: [Getting Started](guide/01-getting-started.md), [Skills](guide/03-skills.md) | Time: ~10 min

## What This Does

A `/reflect` skill that reviews your conversation at the end of a session, identifies learnings, and suggests improvements to your `CLAUDE.md`. Over time, your setup gets better automatically.

## Setup

### Step 1: Create the skill

```bash
mkdir -p .claude/skills/reflect
```

Create `.claude/skills/reflect/SKILL.md`:

```markdown
---
name: reflect
description: Review conversation and suggest CLAUDE.md improvements
---

# /reflect - Session Retrospective

Review this conversation to identify learnings for CLAUDE.md.

## WORKFLOW

### Step 1: Review Conversation
Look through this session for:
- New patterns or conventions we discovered
- Mistakes that could be prevented with better docs
- Missing information that caused confusion or delays
- Ambiguous instructions we had to clarify
- Workarounds we found

### Step 2: Read Current CLAUDE.md
Check what's already documented vs what we needed but didn't have.

### Step 3: Present Suggestions

For each suggestion:

#### [N]. [Section Name]
**Type**: New | Clarification | Correction
**Where**: CLAUDE.md > [Section]
**Add**:
> [Exact text to add or modify]

**Why**: [One sentence explaining why this helps]

### Step 4: Prioritize
Order suggestions by impact: most likely to prevent future confusion first.

## RULES
1. Suggest changes - don't apply them
2. Be specific - include the exact text to add
3. Don't duplicate - check if something is already documented
4. Keep it concise - CLAUDE.md should stay under 500 lines
5. Most impactful suggestions first
```

### Step 2: Use it

At the end of any work session:

```
> /reflect
```

Claude reviews the conversation and produces a list of specific, actionable improvements. Apply the good ones to your `CLAUDE.md`.

## Try It

1. Do a real work session with Claude (debugging, writing code, anything)
2. At the end, run `/reflect`
3. Review the suggestions
4. Apply 1-2 of the best ones to your `CLAUDE.md`
5. Next session, Claude will use the improved context

## The Feedback Loop

```
Work session
    ↓
/reflect at end
    ↓
Review suggestions
    ↓
Apply good ones to CLAUDE.md
    ↓
Next session is better
    ↓
Repeat
```

After 5-10 sessions, your `CLAUDE.md` will be well-tuned to your actual workflow.

## Customize

**Auto-save suggestions**: Add a step that writes suggestions to a file for later review:

```markdown
### Step 5: Save
Write suggestions to `docs/reflections/YYYY-MM-DD.md` for the user to review later.
```

**Focus areas**: If you want `/reflect` to focus on specific things (e.g., only security-related learnings), add a filter:

```markdown
## FOCUS
Only suggest improvements related to: security, error handling, testing patterns.
```
