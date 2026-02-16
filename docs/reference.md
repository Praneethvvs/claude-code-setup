# Reference

Quick-reference for Claude Code concepts, permissions, models, and common errors.

---

## Glossary

| Term | Meaning |
|------|---------|
| **CLAUDE.md** | Markdown file with project instructions. Auto-loaded every conversation. |
| **MCP** | Model Context Protocol - standard for connecting Claude to external tools. |
| **MCP Server** | Process that exposes tools to Claude (e.g., Context7, Playwright). |
| **Skill** | Specialist persona defined in `.claude/skills/<name>/SKILL.md`. Invoked via `/skill-name`. |
| **Command** | One-shot workflow defined in `.claude/commands/<name>.md`. Invoked via `/command-name`. |
| **Subagent** | Child Claude instance launched in parallel for a subtask (via the Task tool). |
| **Plan Mode** | Read-only mode where Claude researches without making changes. |
| **`--print`** | Non-interactive mode: stdin in, stdout out, then exit. Used for automation. |
| **Opus / Sonnet / Haiku** | Claude model tiers: most capable / balanced / fastest. |

---

## File Locations

| File | Purpose | In `.gitignore`? |
|------|---------|-----------------|
| `CLAUDE.md` | Project context + rules | No (share with team) |
| `.claude/settings.json` | Shared project permissions | No |
| `.claude/settings.local.json` | Personal permission overrides | Yes |
| `~/.claude/settings.json` | Global permissions (all projects) | N/A |
| `~/.claude/CLAUDE.md` | Personal instructions (all projects) | N/A |
| `.mcp.json` | MCP server config (may contain keys) | Yes |
| `.mcp.json.template` | MCP config template for team | No |
| `.claudeignore` | Files Claude should skip | No |
| `.claude/commands/*.md` | Command definitions | No |
| `.claude/skills/*/SKILL.md` | Skill definitions | No |

---

## Permission Modes

Set in `settings.local.json` or `settings.json`:

```json
{
  "permissions": {
    "defaultMode": "allowEdits"
  }
}
```

| Mode | Behavior | When to Use |
|------|----------|-------------|
| `"default"` | Asks before every command | Starting out, untrusted projects |
| `"allowEdits"` | Edits files freely, asks before shell commands | Day-to-day development |
| `"bypassPermissions"` | Does everything without asking | Only for fully trusted setups |

---

## Permission Syntax

```json
{
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(python:*)",
      "WebSearch",
      "WebFetch(domain:github.com)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force:*)"
    ]
  }
}
```

| Pattern | Matches |
|---------|---------|
| `Bash(git status:*)` | Any command starting with `git status` |
| `Bash(python:*)` | Any command starting with `python` |
| `WebSearch` | Web search capability |
| `WebFetch(domain:X)` | Fetching pages from domain X |

**Principle**: Pre-approve read-only operations. Leave write/destructive operations requiring manual approval.

---

## Starter Permissions

Copy-paste into `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git branch:*)",
      "Bash(git checkout:*)",
      "Bash(git fetch:*)",
      "Bash(git stash:*)",
      "Bash(git show:*)",

      "Bash(python:*)",
      "Bash(python3:*)",
      "Bash(pytest:*)",
      "Bash(pip install:*)",
      "Bash(ruff:*)",

      "Bash(gh pr:*)",
      "Bash(gh issue:*)",
      "Bash(gh repo:*)",
      "Bash(gh run:*)",

      "Bash(ls:*)",
      "Bash(tree:*)",
      "Bash(find:*)",
      "Bash(grep:*)",
      "Bash(curl:*)",
      "Bash(jq:*)",

      "WebSearch",
      "WebFetch(domain:github.com)",
      "WebFetch(domain:docs.python.org)",
      "WebFetch(domain:stackoverflow.com)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ]
  }
}
```

---

## Models

| Model ID | Tier | Best For |
|----------|------|---------|
| `claude-opus-4-6` | Opus | Complex planning, analysis, deep research |
| `claude-sonnet-4-5-20250929` | Sonnet | Code writing, review, implementation |
| `claude-haiku-4-5-20251001` | Haiku | Simple queries, formatting, triage |

Set in skill frontmatter:
```markdown
---
model: claude-sonnet-4-5-20250929
---
```

---

## Common Errors

| Problem | Cause | Fix |
|---------|-------|-----|
| MCP server not loading | Added after Claude started | Restart Claude (`exit` then `claude`) |
| "Permission denied" on command | Not in `allow` list | Add to `.claude/settings.local.json` allow |
| Claude ignoring CLAUDE.md rules | File too long, rules buried | Move critical rules to top of file |
| Skill not found | Wrong directory structure | Must be `.claude/skills/<name>/SKILL.md` |
| Command not found | Wrong file location | Must be `.claude/commands/<name>.md` |
| `--print` producing no output | No stdin provided | Pipe or redirect input: `echo "task" \| claude --print` |
| Rate limit errors in parallel | Too many concurrent agents | Add `sleep 2` between launches or reduce count |

---

## Resources

- [Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code)
- [MCP Server Directory](https://github.com/modelcontextprotocol/servers)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)
