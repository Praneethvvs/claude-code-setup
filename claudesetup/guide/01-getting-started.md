# Getting Started

> Time: ~15 min | You will learn: **how Claude Code is configured** | You will build: **a working project setup**

## What You're Building

A complete Claude Code configuration for any project: a `CLAUDE.md` that gives Claude context, a permissions file that controls what it can do, and a `.claudeignore` that keeps it focused.

## The Concept

Claude Code reads a layered set of config files from your project:

```
your-project/
├── CLAUDE.md                    # Project context + rules (the most important file)
├── .claude/
│   ├── settings.local.json      # Permissions: what Claude can run without asking
│   ├── commands/                 # Reusable workflows (guide 02)
│   └── skills/                  # Specialist personas (guide 03)
├── .mcp.json                    # External tool connections (guide 04)
└── .claudeignore                # Files Claude should skip
```

| File | Purpose | Analogy |
|------|---------|---------|
| `CLAUDE.md` | Project context, rules, conventions | New-hire orientation doc |
| `settings.local.json` | Which commands Claude can run freely | Access control list |
| `.mcp.json` | Connections to external services | API integrations |
| `commands/` | One-shot workflows triggered by `/name` | Shell scripts |
| `skills/` | Specialist personas with deep domain knowledge | Expert consultants |
| `.claudeignore` | Files Claude should never read | `.gitignore` for AI |

## Exercise

### Step 1: Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### Step 2: Create CLAUDE.md

At your project root, create `CLAUDE.md`:

```markdown
# CLAUDE.md

## Project Overview
[Project name] - [One-line description]

## Tech Stack
- Language: Python 3.11+
- Framework: FastAPI
- Tests: pytest
- Linting: ruff

## Quick Commands
- Install: `pip install -r requirements.txt`
- Test: `pytest`
- Run: `python -m src.main`
- Lint: `ruff check .`

## Project Structure
- `src/` - Application source code
- `tests/` - Test files
- `docs/` - Documentation

## Critical Rules
1. Never commit secrets, API keys, or credentials
2. Never force push without explicit request
3. Never modify production configs without approval
```

**Adapt this to your actual project.** The tech stack, commands, and structure should match what you really use.

### Step 3: Create permissions

```bash
mkdir -p .claude
```

Create `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(python:*)",
      "Bash(pytest:*)",
      "Bash(pip install:*)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ]
  }
}
```

The pattern `Bash(command-prefix:*)` allows any command starting with that prefix. Pre-approve **read-only** and **safe** operations. Leave destructive commands requiring manual approval.

### Step 4: Create .claudeignore

Create `.claudeignore` at your project root:

```
# Dependencies
node_modules/
venv/
.venv/

# Build artifacts
dist/
build/
*.egg-info/

# Large data
*.csv
*.parquet
data/

# Secrets
.env
.env.*
```

This keeps Claude focused on your actual source code instead of wasting context on dependencies and data files.

### Step 5: First run

```bash
cd your-project
claude
```

Claude reads your `CLAUDE.md` automatically. Try asking it something about your project.

## Verify It Works

1. Run `claude` in your project directory
2. Ask: "What project is this and what's the tech stack?"
3. Claude should answer using context from your `CLAUDE.md`
4. Run a git command like `git status` - Claude should execute it without asking permission
5. Ask Claude to `rm -rf /` - it should refuse or ask for confirmation

## What You Learned

- **CLAUDE.md** is the single most important file - it's loaded every conversation
- **Permissions** control what Claude can do without asking (`allow` / `deny` lists)
- **.claudeignore** keeps large/irrelevant files out of Claude's context
- Put **safety rules at the top** of CLAUDE.md - Claude prioritizes content near the top
- Keep CLAUDE.md under **500 lines** - link to detailed docs instead of inlining everything

### Where CLAUDE.md Can Live

| Location | Scope | Use For |
|----------|-------|---------|
| `~/.claude/CLAUDE.md` | All your projects | Personal workflow preferences |
| `project-root/CLAUDE.md` | One project (shared via git) | Project-wide rules |
| `project-root/subdir/CLAUDE.md` | One subdirectory | Module-specific instructions |

Claude merges all levels. More specific files take priority.

---

Next: [Commands](guide/02-commands.md)
