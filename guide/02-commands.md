# Commands

> Time: ~15 min | You will learn: **how commands work** | You will build: **a /compose command for smart git commits**

## What You're Building

A command that analyzes your staged git changes and splits them into logical, atomic commits - grouping features, fixes, and refactors separately.

## The Concept

Commands are one-shot workflows you trigger with `/command-name`. Each command is a single markdown file in `.claude/commands/`.

```
.claude/commands/
├── compose.md              # Invoked as /compose
├── test.quick.md           # Invoked as /test.quick
└── deploy.staging.md       # Invoked as /deploy.staging
```

A command file has frontmatter (metadata) and a body (instructions):

```markdown
---
description: "What this command does"
allowed-tools:
  - "Bash(git *)"
argument-hint: "[expected arguments]"
---

# The instructions Claude follows when you invoke this command

$ARGUMENTS
```

- `description` appears when Claude lists available commands
- `allowed-tools` grants additional permissions for this command only
- `$ARGUMENTS` is replaced with whatever you type after the command name

**Commands vs Skills**: Commands are simple, one-shot workflows (like a shell script). Skills (next guide) are specialist personas with deeper domain knowledge and multiple modes.

## Exercise

### Step 1: Create the command file

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
   - Stage only the files/hunks for this group
   - Commit with conventional message: `type(scope): description`
4. Show all created commits with `git log --oneline`

## Commit Types
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Code restructuring (no behavior change)
- `test` - Adding or updating tests
- `docs` - Documentation changes
- `chore` - Build, config, tooling changes

## Rules
- Each commit should be independently reviewable
- Each commit should build successfully
- If changes are truly inseparable, explain why and make one commit
- Never include unrelated changes in the same commit

$ARGUMENTS
```

### Step 2: Add git permissions

In `.claude/settings.local.json`, make sure your `allow` list includes:

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

### Step 3: Try it

```bash
# Make some changes across multiple files
git add -A

# Start Claude and run the command
claude
> /compose
```

Claude will analyze your staged diff, group the changes by concern, and create separate commits for each group.

## Verify It Works

1. Stage a mix of changes (e.g., a bug fix + a new test + a README update)
2. Run `/compose` in Claude
3. Check `git log --oneline` - you should see multiple atomic commits
4. Each commit message should follow the `type(scope): description` format

## What You Learned

- Commands live in `.claude/commands/` as markdown files
- Each file = one command, invoked as `/filename` (without `.md`)
- `$ARGUMENTS` passes user input into the command
- `allowed-tools` in frontmatter grants extra permissions for that command
- Dots in filenames create namespaced commands: `test.quick.md` becomes `/test.quick`

---

Next: [Skills](guide/03-skills.md)
