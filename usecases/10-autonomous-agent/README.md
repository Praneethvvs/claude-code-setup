# 10: Autonomous Agent

## What It Does

Runs Claude Code inside Docker as a headless agent that completes tasks without human interaction. Feed it a task template, it produces output files.

Use cases:
- Nightly code quality reports
- Automated security audits
- Test result summaries
- Data processing pipelines
- CI/CD-triggered code review

## Why It Matters

Some tasks are well-defined enough that they don't need interactive back-and-forth. Automating them frees up developer time.

## Prerequisites

- Docker installed
- Claude Code API access (ANTHROPIC_API_KEY)

## Setup Steps

### 1. Simple Version (No Docker)

Use Claude Code's `--print` mode for non-interactive execution:

```bash
# Run a task, get output
echo "Review all TODO comments in src/ and create a prioritized list" | claude --print > todo-report.md

# Run with a specific prompt file
claude --print < prompts/security-audit.md > reports/security-audit.md
```

### 2. Template System

Create task templates with `{{VARIABLE}}` placeholders:

```bash
mkdir -p templates
```

Create `templates/code-review.md`:

```markdown
# Code Review: {{FILE_PATH}}

Review the file at {{FILE_PATH}} against these standards:
1. Type hints on all functions
2. Proper error handling
3. No hardcoded secrets
4. Test coverage exists
5. Async for I/O operations

Output a structured review with severity levels (CRITICAL/HIGH/MEDIUM/LOW).
```

Create `scripts/run-template.sh`:

```bash
#!/bin/bash
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

Usage:

```bash
chmod +x scripts/run-template.sh
./scripts/run-template.sh code-review FILE_PATH=src/api/routes/users.py
```

### 3. Docker Version (Full Isolation)

Create `Dockerfile.claude`:

```dockerfile
FROM node:20-slim
RUN npm install -g @anthropic-ai/claude-code
WORKDIR /workspace
COPY CLAUDE.md .
COPY .claude/ .claude/
ENTRYPOINT ["claude", "--print"]
```

```bash
# Build
docker build -f Dockerfile.claude -t {{org}}-claude-agent .

# Run
echo "Analyze src/ for security vulnerabilities" | \
  docker run -i -v $(pwd)/src:/workspace/src {{org}}-claude-agent > report.md
```

### 4. CI/CD Integration

GitHub Actions example:

```yaml
name: AI Code Review
on: pull_request

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code
      - name: Run AI Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          echo "Review the changes in this PR for bugs and security issues. Files changed: $(git diff --name-only origin/main)" | claude --print > review.md
      - name: Post Review
        run: gh pr comment ${{ github.event.number }} --body-file review.md
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## See Also

- [explanation.md](explanation.md) - Docker configuration, template patterns, autonomous agent design
