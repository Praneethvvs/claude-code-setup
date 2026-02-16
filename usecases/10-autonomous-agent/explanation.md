# Explanation: Autonomous Agent

## How the Original Enterprise Setup Worked

A production setup can include a full autonomous agent system with this structure:

```
claude-agent/
├── .claude/
│   ├── commands/
│   └── skills/reflect/SKILL.md
├── .mcp.json
├── templates/           # Task templates with {{VARIABLE}} placeholders
├── scripts/
│   └── run-template.sh  # Template runner
└── Dockerfile
```

### Key Design Decisions

**Docker isolation**: The agent runs in a container with:
- The codebase mounted as a read-only volume
- Its own CLAUDE.md and skills (copied, not symlinked - Docker can't follow symlinks outside build context)
- Its own MCP configuration
- Output written to a mounted volume

**Template system**: Tasks are defined as markdown templates:
```markdown
# Task: Security Audit for {{REPO}}
## Scope: {{SCOPE}}

1. Analyze {{REPO}} for OWASP Top 10 vulnerabilities
2. Check for hardcoded secrets
3. Review dependency versions
4. Output report to: output/audit-{{REPO}}-{{DATE}}.md
```

Run with: `./scripts/run-template.sh audit --var REPO=api --var SCOPE=auth`

**Skills portability**: Skills had to be explicitly copied into the Docker context. The original setup had a note: "Docker doesn't follow symlinks outside build context." This is a common gotcha.

### What They Used It For

- **Automated code reviews** on PRs (triggered by CI/CD)
- **Nightly security scans** across repositories
- **Report generation** (sprint summaries, test coverage reports)
- **Batch processing** (processing multiple tickets, files, or repos)

## Claude Features That Enable This

### `claude --print` Mode

The `--print` flag runs Claude non-interactively:
- Reads from stdin
- Writes to stdout
- No interactive prompts
- Returns exit code on completion

This makes Claude scriptable, like any other CLI tool.

### Skills in Containers

Skills work inside Docker, but with caveats:
- Copy skill files into the container (don't symlink)
- Copy CLAUDE.md into the container
- MCP servers need network access from inside the container
- Environment variables (API keys) must be passed to the container

### MCP Servers in Docker

MCP servers that are themselves Docker containers need Docker-in-Docker or host Docker socket access:

```bash
docker run -i \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/workspace \
  {{org}}-claude-agent
```

Or use non-Docker MCP servers (Node.js, Python) that run inside the container.

## Adapting for {{ORG}}

### Start Simple

Don't containerize on day one. Start with `claude --print`:

```bash
# Daily TODO report
echo "Find all TODO and FIXME comments in src/. Group by file. Prioritize by importance." | claude --print > reports/todos-$(date +%Y-%m-%d).md

# Pre-commit review
git diff --staged | claude --print "Review these staged changes for bugs, security issues, and style violations"
```

### Then Add Templates

Once you have recurring tasks, create templates:

```
templates/
├── code-review.md      # Review a specific file
├── security-audit.md   # Audit a service for vulnerabilities
├── test-report.md      # Summarize test results
└── migration-check.md  # Verify database migration safety
```

### Then Add CI/CD

When you trust the output, add to your pipeline:

```yaml
# .github/workflows/ai-review.yml
on: pull_request
jobs:
  ai-review:
    steps:
      - run: echo "Review PR changes" | claude --print > review.md
      - run: gh pr comment $PR --body-file review.md
```

### Cost Considerations

Each autonomous run consumes API tokens. For cost control:
- Use Sonnet or Haiku for simple, repetitive tasks
- Use Opus only for complex analysis
- Set token limits in your prompts
- Monitor costs via the Anthropic dashboard
