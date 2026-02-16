# 06: Smart Git Commit Composition

## What It Does

You stage a bunch of changes. Claude analyzes them, groups by logical concern (feature/fix/refactor/test), and creates multiple atomic commits - even splitting changes within the same file.

## Why It Matters

- Atomic commits are easier to review, revert, and bisect
- Good commit messages explain "why", not just "what"
- No more 500-line monolithic commits mixing features with fixes

## Prerequisites

None. Just git.

## Setup Steps

### 1. Create the command

```bash
mkdir -p .claude/commands
```

Create `.claude/commands/compose.md`:

```markdown
---
description: "Analyze staged changes and split into logical atomic commits"
allowed-tools:
  - "Bash(git *)"
argument-hint: "[optional: commit strategy or prefix]"
---

# Compose Multiple Commits

Analyze all staged changes and split into logical, atomic commits.

## Process

1. Run `git diff --staged` to see all staged changes
2. Analyze and group by logical concern:
   - Feature additions
   - Bug fixes
   - Refactoring
   - Tests
   - Documentation
   - Configuration
3. For each group:
   - `git reset` to unstage everything
   - For files belonging entirely to this group: `git add <file>`
   - For files with mixed changes: create a patch with only relevant hunks, apply with `git apply --cached`
   - Commit with conventional message: `type(scope): description`
4. Show all created commits with `git log --oneline`

## Commit Types
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Code restructuring (no behavior change)
- `test` - Adding or updating tests
- `docs` - Documentation changes
- `chore` - Build, config, tooling changes
- `perf` - Performance improvement

## Rules
- Each commit should be independently reviewable
- Each commit should build successfully
- If changes are truly inseparable, explain why and make one commit
- Never include unrelated changes in the same commit

$ARGUMENTS
```

### 2. Add git permissions

In `.claude/settings.local.json`, ensure these are in `allow`:

```json
"Bash(git add:*)",
"Bash(git commit:*)",
"Bash(git diff:*)",
"Bash(git log:*)",
"Bash(git status:*)",
"Bash(git reset:*)",
"Bash(git stash:*)",
"Bash(git apply:*)"
```

### 3. Use it

```bash
# Stage your changes
git add -A

# Let Claude compose the commits
claude
> /compose
```

## See Also

- [explanation.md](explanation.md) - Hunk-level splitting, the patch technique
