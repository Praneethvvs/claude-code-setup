# 09: Self-Improving Configuration

## What It Does

At the end of a session, Claude reviews the entire conversation, identifies learnings, and suggests improvements to your CLAUDE.md. Over time, your setup gets better automatically.

## Why It Matters

- Tribal knowledge gets captured instead of lost between sessions
- Mistakes that happen once get documented so they don't happen twice
- New patterns and conventions are recorded as they emerge
- Ambiguous instructions get clarified

## Prerequisites

None.

## Setup Steps

### 1. Create the skill

```bash
mkdir -p .claude/skills/reflect
```

Create `.claude/skills/reflect/SKILL.md`:

```markdown
---
name: reflect
description: Review conversation and suggest CLAUDE.md improvements
allowed-tools: Read, Glob, Grep, Bash
---

# /reflect - Session Retrospective

Review this conversation to identify learnings for CLAUDE.md.

## Steps

### 1. Review Conversation History
Look through this session for:
- New patterns or conventions we discovered
- Mistakes that could be prevented with better docs
- Missing information that caused confusion or delays
- Ambiguous instructions we had to clarify
- New tools or integrations we used
- Workarounds we found

### 2. Read Current CLAUDE.md
Check what's already documented vs what we needed.

### 3. Explore Docs Structure
Look at existing documentation to find the best home for suggestions.

### 4. Present Suggestions
For each:

#### [N]. [Section]
**Type**: New | Clarification | Correction
**Where**: CLAUDE.md > [Section] (or new file)
**Suggestion**:
> [Exact text to add or modify]

**Rationale**: [Why this helps]

### 5. Save
Ask if user wants to save suggestions to:
`docs/claude-improvements/{YYYY-MM-DD}-{topic}.md`

## RULES
1. Read-only - suggest changes, don't apply them
2. Be specific - include exact text to add
3. Include rationale - explain why each suggestion helps
4. Don't duplicate - check if something is already documented
5. Prioritize - most impactful suggestions first
```

### 2. Allow the reflect skill

In `.claude/settings.local.json`:

```json
"Skill(reflect)"
```

### 3. Use it

At the end of any session:

```
> /reflect
```

### 4. Apply the Good Ones

Review Claude's suggestions, apply what makes sense to your CLAUDE.md, discard the rest.

## The Feedback Loop

```
Work session
    ↓
/reflect at end
    ↓
Review suggestions (you)
    ↓
Apply good ones to CLAUDE.md
    ↓
Next session is better
    ↓
Repeat
```

After 5-10 sessions, your CLAUDE.md will be remarkably tuned to your actual workflow.

## See Also

- [explanation.md](explanation.md) - How the original setup recovered full context, transcript reading
