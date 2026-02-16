# Automation

> Time: ~15 min | You will learn: **how to run Claude headless** | You will build: **a template system + CI/CD integration**

## What You're Building

A way to run Claude non-interactively: pipe in a task, get output. Then a reusable template system for common tasks. Then a GitHub Actions integration for automated code review on PRs.

## The Concept

Claude Code has a `--print` flag that runs it non-interactively. Feed it a prompt via stdin, it produces output on stdout, then exits. No back-and-forth conversation.

```bash
echo "Summarize the README" | claude --print
```

This unlocks automation: scripts, cron jobs, CI/CD pipelines, and any workflow where you want Claude to process something without a human in the loop.

## Exercise

### Step 1: Basic --print usage

```bash
# Simple one-liner
echo "List all TODO comments in src/" | claude --print

# Save output to a file
echo "Review src/api/routes.py for security issues" | claude --print > review.md

# Pipe a file as the prompt
claude --print < prompts/security-audit.md > reports/security-audit.md
```

### Step 2: Build a template system

Templates let you define reusable prompts with `{{VARIABLE}}` placeholders.

Create `templates/code-review.md`:

```markdown
# Code Review: {{FILE_PATH}}

Review the file at {{FILE_PATH}} against these standards:
1. Type hints on all functions
2. Proper error handling
3. No hardcoded secrets
4. Test coverage exists
5. Async for I/O operations

Output a structured review with severity levels (CRITICAL / HIGH / MEDIUM / LOW).
```

Create `scripts/run-template.sh`:

```bash
#!/bin/bash
# Usage: ./scripts/run-template.sh <template-name> KEY=VALUE KEY2=VALUE2
TEMPLATE=$1
shift

CONTENT=$(cat "templates/$TEMPLATE.md")
while [[ $# -gt 0 ]]; do
    KEY="${1%%=*}"
    VALUE="${1#*=}"
    CONTENT="${CONTENT//\{\{$KEY\}\}/$VALUE}"
    shift
done

echo "$CONTENT" | claude --print
```

```bash
chmod +x scripts/run-template.sh
```

Use it:

```bash
./scripts/run-template.sh code-review FILE_PATH=src/api/routes/users.py
```

### Step 3: CI/CD integration

Add automated code review to your pull requests with GitHub Actions.

Create `.github/workflows/ai-review.yml`:

```yaml
name: AI Code Review
on: pull_request

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run AI Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          CHANGED=$(git diff --name-only origin/main)
          echo "Review these changed files for bugs, security issues, and style problems: $CHANGED" \
            | claude --print > review.md

      - name: Post Review Comment
        run: gh pr comment ${{ github.event.number }} --body-file review.md
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Setup**: Add your `ANTHROPIC_API_KEY` as a repository secret in GitHub Settings > Secrets and variables > Actions.

## Verify It Works

1. Run `echo "What is 2+2?" | claude --print` - you should get a response on stdout
2. Create a template and run it with `run-template.sh` - verify variable substitution works
3. For CI/CD: push the workflow file, open a PR, and check for the review comment

## What You Learned

- `claude --print` runs Claude non-interactively (stdin in, stdout out)
- Templates with `{{VARIABLE}}` placeholders make prompts reusable
- A simple shell script can substitute variables and pipe to Claude
- GitHub Actions can run Claude on every PR for automated review
- The key to CI/CD is the `ANTHROPIC_API_KEY` secret

---

Next: [Parallel Agents](guide/06-parallel-agents.md)
