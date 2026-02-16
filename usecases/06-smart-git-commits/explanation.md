# Explanation: Smart Git Commit Composition

## How the Original Enterprise Setup Worked

The `/compose` command was one of the simplest but most-used features. Its key innovation was **hunk-level splitting**: when a single file contains changes belonging to different logical commits, Claude creates temporary patch files with only the relevant diff hunks.

### The Patch Technique

```bash
# 1. Unstage everything
git reset

# 2. For a file with mixed changes, extract specific hunks
git diff -- src/utils.py > /tmp/full-diff.patch
# Claude edits the patch to keep only auth-related hunks
# (removes irrelevant hunk headers and content)

# 3. Apply only the selected hunks
git apply --cached /tmp/auth-hunks.patch

# 4. Commit
git commit -m "fix(auth): handle expired session tokens"

# 5. Clean up
rm /tmp/auth-hunks.patch

# 6. Stage remaining changes for next commit
git add src/utils.py
git commit -m "refactor(utils): extract validation helpers"
```

This is something most developers never do manually because it's tedious. Claude automates it perfectly.

### Commit Message Convention

The setup used conventional commits: `type(scope): description`

```
fix(auth): handle expired session tokens during form submission
refactor(api): extract validation into middleware
feat(dashboard): add real-time notification counter
test(auth): add missing tests for token expiry
```

## Claude Features That Enable This

### Commands

A single markdown file with clear instructions. Commands are ideal for this because:
- The workflow is always the same (analyze, group, commit)
- No deep domain knowledge needed (unlike skills)
- `$ARGUMENTS` lets you pass optional instructions (e.g., `/compose prefix with {{ORG}}-42`)

### Bash Permissions

Pre-approved git commands make the flow seamless. Without them, Claude asks for approval at every `git add`, `git commit`, etc.

### The `allowed-tools` Restriction

```yaml
allowed-tools:
  - "Bash(git *)"
```

The command restricts itself to only git operations. This prevents accidental side effects.

## Adapting for {{ORG}}

### Match Your Commit Convention

If {{ORG}} uses ticket-prefixed commits, update the command:

```markdown
## Commit Format
- Format: `{{ORG}}-{ticket}: {type}: {description}`
- Example: `{{ORG}}-42: feat: add user export endpoint`
- If no ticket context: `chore: {description}`
```

### Add to CLAUDE.md

```markdown
## Git Conventions
- Commit format: `{{ORG}}-{ticket}: {type}: {description}`
- Types: feat, fix, refactor, test, docs, chore
- Branch: `feature/{{ORG}}-{ticket}-{brief-name}`
```

Claude will follow these conventions when composing commit messages.

### Combine with PR Creation

After composing commits, you can ask Claude to create the PR:

```
> /compose
[creates 3 atomic commits]

> Create a PR for these changes
[Claude runs gh pr create with a summary of all commits]
```
