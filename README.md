# Claude Code Setup Guide

A practical, beginner-friendly guide to setting up Claude Code for any organization. Covers Python + Azure + AWS + GitHub + Azure DevOps, but the patterns apply to any stack.

> **How to use this guide**: Replace `{{ORG}}` with your organization name and `{{org}}` with the lowercase version throughout.

---

## Table of Contents

1. [What is Claude Code?](#what-is-claude-code)
2. [Core Concepts](#core-concepts)
3. [Quick Start (5 Minutes)](#quick-start-5-minutes)
4. [CLAUDE.md - The Brain](#claudemd---the-brain)
5. [Permissions - The Access Control](#permissions---the-access-control)
6. [MCP Servers - The Plugins](#mcp-servers---the-plugins)
7. [Skills - The Specialists](#skills---the-specialists)
8. [Commands - The Workflows](#commands---the-workflows)
9. [Best Practices](#best-practices)
10. [Use Cases](#use-cases)
11. [Glossary](#glossary)

---

## What is Claude Code?

Claude Code is Anthropic's CLI tool that puts Claude AI directly in your terminal. It reads your codebase, runs commands, edits files, and connects to external services. Think of it as an AI team member that lives in your terminal.

### Install

```bash
npm install -g @anthropic-ai/claude-code
```

### Run

```bash
cd your-project
claude
```

That's it. Claude reads your project files and starts helping.

---

## Core Concepts

Claude Code uses a layered config system. Here's the full picture:

```
your-project/
├── CLAUDE.md                    # Brain: project context + rules
├── .claude/
│   ├── settings.local.json      # Permissions: what Claude can/can't run
│   ├── commands/                # Commands: reusable workflows (/command-name)
│   └── skills/                  # Skills: specialist personas (/skill-name)
├── .mcp.json                    # Plugins: external tool connections
└── .claudeignore                # Ignore: files Claude shouldn't read
```

| File | Purpose | Analogy |
|------|---------|---------|
| `CLAUDE.md` | Project context, rules, conventions | New-hire orientation doc |
| `settings.local.json` | Which commands Claude can run without asking | Access control list |
| `.mcp.json` | Connections to JIRA, Slack, databases, etc. | API integrations |
| `commands/` | One-shot workflows triggered by `/name` | Shell scripts |
| `skills/` | Specialist personas with deep domain knowledge | Expert consultants |
| `.claudeignore` | Files Claude should never read | `.gitignore` for AI |

---

## Quick Start (5 Minutes)

### Step 1: Create CLAUDE.md

At your project root:

```markdown
# CLAUDE.md

## Project Overview
[Project name] - [One-line description]
Organization: {{ORG}}

## Tech Stack
- Language: Python 3.11+
- Cloud: Azure (primary), AWS (secondary)
- Repos: GitHub + Azure DevOps
- CI/CD: GitHub Actions / Azure Pipelines

## Quick Commands
- Install: `pip install -r requirements.txt`
- Test: `pytest`
- Run: `python -m src.main`
- Lint: `ruff check .`

## Project Structure
- `src/` - Application source code
- `tests/` - Test files
- `docs/` - Documentation
- `infra/` - Infrastructure as code

## Critical Rules
1. Never commit secrets, API keys, or credentials
2. Never force push without explicit request
3. Never modify production configs without approval
```

### Step 2: Create permissions file

```bash
mkdir -p .claude
```

Create `.claude/settings.local.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(python:*)",
      "Bash(python3:*)",
      "Bash(pytest:*)",
      "Bash(pip install:*)",
      "Bash(pip3 install:*)"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ]
  }
}
```

### Step 3: Run Claude

```bash
cd your-project
claude
```

Claude will read your CLAUDE.md automatically and follow those rules.

---

## CLAUDE.md - The Brain

This is the single most important file. It's loaded into Claude's context every conversation. Everything Claude needs to know about your project goes here.

### Where to Place It

| Location | Scope | Use For |
|----------|-------|---------|
| `~/.claude/CLAUDE.md` | All your projects (personal) | Your personal workflow preferences |
| `project-root/CLAUDE.md` | One project (shared via git) | Project-wide rules and context |
| `project-root/subdir/CLAUDE.md` | One subdirectory | Module-specific instructions |

Claude merges all levels. More specific files take priority.

### What to Include

#### 1. Critical Rules (Top of File)

Claude prioritizes content near the top. Put safety rules first:

```markdown
## Critical Rules (Never Override)

1. **Secrets**: Never commit API keys, tokens, or credentials to any repository
2. **Destructive Git**: Never force push without explicit request
3. **Production**: Never modify production configs without approval
4. **Data**: Never log PII or sensitive customer data
5. **Plan Mode**: When in plan mode, gather information only - no edits
```

#### 2. Behavioral Directives

```markdown
## Behavioral Directives

### Response Style
- Be concise
- Present data directly in chat - don't hide in collapsed outputs
- When suggesting CLI commands, verify they're installed first

### Question vs Action
- When I ask "why/what/how" - explain first, don't take action
- Ask before destructive changes, multi-file edits, or breaking changes
- Don't assume my intent - ask clarifying questions when unclear

### Task Focus
- Recap original goals during long conversations
- Flag when discussion drifts off-topic
- Summarize at milestones: what's done, what's next
```

#### 3. Required Reading (Context Chain)

Don't cram everything into CLAUDE.md. Point Claude to other docs:

```markdown
## Required Reading (Read Before Acting)

| Task | Read First |
|------|-----------|
| Azure operations | docs/azure-guide.md |
| AWS operations | docs/aws-guide.md |
| Database changes | docs/database-conventions.md |
| API changes | docs/api-standards.md |
| CI/CD changes | docs/pipeline-guide.md |
| Git operations | docs/git-conventions.md |
```

#### 4. Project Architecture

```markdown
## Architecture

### Services
| Service | Purpose | Port | Repo |
|---------|---------|------|------|
| API Gateway | REST API | 8000 | github.com/{{org}}/api |
| Worker | Background jobs | - | github.com/{{org}}/worker |
| Frontend | Web UI | 3000 | github.com/{{org}}/web |

### Key Patterns
- FastAPI for all REST services
- Pydantic models for validation
- SQLAlchemy + Alembic for database
- pytest for testing
- Ruff for linting
```

#### 5. Development Workflow

```markdown
## Git Conventions

- Branch: `feature/{ticket-id}-{brief-description}`
- Commit: `{ticket-id}: {description}`
- PR: Always link to ticket, require at least 1 review

## Common Issues

| Issue | Solution |
|-------|---------|
| Import errors after pull | `pip install -r requirements.txt` |
| Port 8000 in use | `lsof -i :8000` then `kill -9 <PID>` |
| Alembic head mismatch | `alembic heads` then merge |
| Azure CLI not logged in | `az login` |
```

#### 6. Environment Info

```markdown
## Environments

| Env | URL | Azure Subscription | AWS Account |
|-----|-----|-------------------|-------------|
| Dev | dev.{{org}}.com | {{org}}-dev | 123456789 |
| Staging | staging.{{org}}.com | {{org}}-staging | 234567890 |
| Prod | {{org}}.com | {{org}}-prod | 345678901 |

## Key Vault / Secrets
- Azure: Key Vault `kv-{{org}}-{env}`
- AWS: Secrets Manager `{{org}}/{env}/*`
```

### Keep It Under 500 Lines

Claude reads the entire file every conversation. If it's bloated, you waste context. Link to detailed docs instead.

---

## Permissions - The Access Control

Controls what Claude can run without asking for approval each time.

### File Locations

| File | Scope | Committed to Git? |
|------|-------|--------------------|
| `.claude/settings.json` | Project (shared with team) | Yes |
| `.claude/settings.local.json` | Project (personal overrides) | No - add to .gitignore |
| `~/.claude/settings.json` | Global (all your projects) | No |

### Permission Modes

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
| `"bypassPermissions"` | Does everything without asking | Only for trusted setups you fully control |

### {{ORG}} Recommended Permissions

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
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
      "Bash(git remote:*)",

      "Bash(python:*)",
      "Bash(python3:*)",
      "Bash(pytest:*)",
      "Bash(pip install:*)",
      "Bash(pip3 install:*)",
      "Bash(source venv/bin/activate)",
      "Bash(ruff:*)",
      "Bash(mypy:*)",
      "Bash(alembic:*)",

      "Bash(az account list:*)",
      "Bash(az account show:*)",
      "Bash(az account set:*)",
      "Bash(az group list:*)",
      "Bash(az keyvault list:*)",
      "Bash(az keyvault secret list:*)",
      "Bash(az keyvault secret show:*)",
      "Bash(az webapp list:*)",
      "Bash(az webapp show:*)",
      "Bash(az webapp config appsettings list:*)",
      "Bash(az monitor log-analytics query:*)",
      "Bash(az monitor app-insights query:*)",
      "Bash(az pipelines:*)",
      "Bash(az repos list:*)",
      "Bash(az repos pr show:*)",
      "Bash(az repos pr diff:*)",
      "Bash(az aks get-credentials:*)",
      "Bash(az aks list:*)",
      "Bash(az acr list:*)",

      "Bash(aws sts get-caller-identity:*)",
      "Bash(aws s3 ls:*)",
      "Bash(aws logs filter-log-events:*)",
      "Bash(aws logs describe-log-groups:*)",
      "Bash(aws ecs list-services:*)",
      "Bash(aws ecs describe-services:*)",
      "Bash(aws ecs describe-tasks:*)",
      "Bash(aws secretsmanager list-secrets:*)",
      "Bash(aws cloudformation describe-stacks:*)",
      "Bash(aws lambda list-functions:*)",

      "Bash(docker ps:*)",
      "Bash(docker compose:*)",
      "Bash(docker logs:*)",
      "Bash(docker build:*)",

      "Bash(kubectl get:*)",
      "Bash(kubectl describe:*)",
      "Bash(kubectl logs:*)",
      "Bash(kubectl config:*)",

      "Bash(gh pr:*)",
      "Bash(gh issue:*)",
      "Bash(gh repo:*)",
      "Bash(gh run:*)",

      "Bash(curl:*)",
      "Bash(jq:*)",
      "Bash(ls:*)",
      "Bash(tree:*)",
      "Bash(find:*)",
      "Bash(grep:*)",
      "Bash(wc:*)",
      "Bash(head:*)",
      "Bash(tail:*)",

      "WebFetch(domain:github.com)",
      "WebFetch(domain:docs.microsoft.com)",
      "WebFetch(domain:docs.aws.amazon.com)",
      "WebSearch"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ]
  },
  "enableAllProjectMcpServers": true,
  "outputStyle": "concise"
}
```

**Pattern syntax**: `Bash(command-prefix:*)` allows any command starting with that prefix.

**Key principle**: Pre-approve **read-only** operations. Leave write operations (deploy, delete, config changes) requiring manual approval.

---

## MCP Servers - The Plugins

MCP (Model Context Protocol) servers give Claude access to external services. They're small processes that expose tools Claude can call.

### Config File: `.mcp.json`

Place at project root. Add to `.gitignore` (contains secrets). Share a `.mcp.json.template` with the team.

```json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["path/to/server.js"],
      "env": {
        "API_KEY": "your-key-here"
      }
    }
  }
}
```

### Recommended MCP Servers for {{ORG}}

#### JIRA + Confluence (Atlassian)

```json
{
  "mcp-atlassian": {
    "command": "docker",
    "args": [
      "run", "-i", "--rm",
      "-e", "CONFLUENCE_URL", "-e", "CONFLUENCE_USERNAME", "-e", "CONFLUENCE_API_TOKEN",
      "-e", "JIRA_URL", "-e", "JIRA_USERNAME", "-e", "JIRA_API_TOKEN",
      "ghcr.io/sooperset/mcp-atlassian:latest"
    ],
    "env": {
      "CONFLUENCE_URL": "https://{{org}}.atlassian.net/wiki",
      "CONFLUENCE_USERNAME": "you@{{org}}.com",
      "CONFLUENCE_API_TOKEN": "YOUR_TOKEN",
      "JIRA_URL": "https://{{org}}.atlassian.net",
      "JIRA_USERNAME": "you@{{org}}.com",
      "JIRA_API_TOKEN": "YOUR_TOKEN"
    }
  }
}
```

Get your token: https://id.atlassian.com/manage-profile/security/api-tokens

#### Browser Automation (Playwright)

```json
{
  "playwright": {
    "command": "npx",
    "args": [
      "@playwright/mcp@latest",
      "--headless", "--browser", "chrome",
      "--viewport-size", "1920x1080"
    ]
  }
}
```

#### Web Research (Perplexity)

```json
{
  "perplexity-ask": {
    "command": "docker",
    "args": [
      "run", "-i", "--rm",
      "-e", "PERPLEXITY_API_KEY", "-e", "PERPLEXITY_ASK_MODEL",
      "mcp/perplexity-ask"
    ],
    "env": {
      "PERPLEXITY_API_KEY": "pplx-YOUR_KEY",
      "PERPLEXITY_ASK_MODEL": "sonar-pro"
    }
  }
}
```

#### Extended Thinking (Sequential Thinking)

```json
{
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  }
}
```

#### Up-to-Date Library Docs (Context7)

```json
{
  "Context7": {
    "command": "npx",
    "args": ["-y", "@upstash/context7-mcp@latest"]
  }
}
```

### Security: Template Pattern

Never commit `.mcp.json` with real keys. Commit a template instead:

```bash
echo ".mcp.json" >> .gitignore
```

Create `.mcp.json.template` (committed):
```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "env": {
        "JIRA_URL": "https://{{org}}.atlassian.net",
        "JIRA_USERNAME": "{{YOUR_EMAIL}}",
        "JIRA_API_TOKEN": "{{YOUR_ATLASSIAN_TOKEN}}"
      }
    }
  }
}
```

After first setup, restart Claude Code (`exit` then `claude`) to load MCP servers.

---

## Skills - The Specialists

Skills turn Claude into a domain expert. They define a persona, workflow, rules, and output format.

### File Structure

```
.claude/skills/
├── SKILL_DEVELOPMENT.md          # Guide for creating new skills
├── my-skill/
│   └── SKILL.md                  # Skill definition
└── another-skill/
    └── SKILL.md
```

### Skill Template

```markdown
---
name: skill-name
description: What this skill does (shown in skill list)
model: claude-sonnet-4-5-20250929
---

# /skill-name - Title

**Purpose**: One-line summary.

## Input
- `/skill-name <arg>` - description

---

## WORKFLOW

### Step 1: Gather Context
[What to read/search]

### Step 2: Analyze
[What to do with the info]

### Step 3: Output
[What to produce]

---

## OUTPUT FORMAT

```markdown
## Result: [title]
[Structured template]
```

---

## RULES
1. **Safety rule** - description
2. **Quality rule** - description

---

## TRIGGER PHRASES
- `/skill-name`
- "natural language trigger"
```

### Model Selection

| Model | Use For | Speed | Cost |
|-------|---------|-------|------|
| `claude-opus-4-6` | Complex planning, analysis, deep research | Slow | High |
| `claude-sonnet-4-5-20250929` | Code writing, review, implementation | Medium | Medium |
| `claude-haiku-4-5-20251001` | Simple queries, formatting, triage | Fast | Low |

---

## Commands - The Workflows

Commands are simpler than skills - one-shot workflows triggered by `/command-name`.

### File Structure

```
.claude/commands/
├── my-command.md
└── grouped.sub-command.md    # Creates /grouped.sub-command
```

### Command Template

```markdown
---
description: "What this command does"
allowed-tools:
  - "Bash(git *)"
argument-hint: "[expected args]"
---

# Command Title

## Process
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Rules
- [Guardrail 1]
- [Guardrail 2]

$ARGUMENTS
```

`$ARGUMENTS` is replaced with whatever the user types after the command name.

---

## Best Practices

### 1. Start Simple, Add Complexity Gradually

Day 1: `CLAUDE.md` + basic permissions.
Week 2: Add MCP servers for tools you use daily.
Month 2: Create skills and commands for repeated workflows.

### 2. Keep CLAUDE.md Under 500 Lines

Link to detailed docs instead of inlining everything.

### 3. Safety First

Always deny destructive commands:

```json
{
  "deny": [
    "Bash(rm -rf *)",
    "Bash(git push --force:*)",
    "Bash(git reset --hard:*)",
    "Bash(drop database:*)"
  ]
}
```

### 4. Use Required Reading for Deep Context

```markdown
## Required Reading
| Task | Read First |
|------|-----------|
| Azure ops | docs/azure-guide.md |
```

### 5. Version Your CLAUDE.md

```markdown
**Version**: 1.0
**Last Updated**: 2026-02-15
```

### 6. Use /reflect to Continuously Improve

End sessions with: "What should we add to CLAUDE.md?"

### 7. Template for Team Onboarding

- `settings.json.template` - permissions with placeholders
- `.mcp.json.template` - MCP config with `{{PLACEHOLDER}}`
- `CLAUDE.md` - committed to repo, shared with everyone

### 8. Separate Concerns

| Concern | Where |
|---------|-------|
| Project context & rules | `CLAUDE.md` |
| Permissions | `.claude/settings.local.json` |
| External integrations | `.mcp.json` |
| Reusable workflows | `.claude/commands/` |
| Domain expertise | `.claude/skills/` |
| Files to ignore | `.claudeignore` |

---

## Use Cases

Real-world problems Claude Code can solve. Each is documented in `usecases/`:

| # | Use Case | Key Feature |
|---|----------|-------------|
| 01 | [Automated Dev Environment Setup](usecases/01-automated-dev-setup/) | Skills + Bash permissions |
| 02 | [Debugging Cloud Deployment Logs](usecases/02-debugging-cloud-logs/) | Azure/AWS CLI permissions |
| 03 | [JIRA/Confluence Automation](usecases/03-jira-confluence-automation/) | MCP Servers (Atlassian) |
| 04 | [Product Spec to JIRA Pipeline](usecases/04-product-spec-pipeline/) | Commands |
| 05 | [AI Story Point Estimation](usecases/05-story-point-estimation/) | Skills + MCP |
| 06 | [Smart Git Commit Composition](usecases/06-smart-git-commits/) | Commands |
| 07 | [Code Expert / PR Review](usecases/07-code-expert-pr-review/) | Skills |
| 08 | [Notification Triage Automation](usecases/08-email-triage/) | Skills + APIs |
| 09 | [Self-Improving Config](usecases/09-self-improving-config/) | Skills (reflect) |
| 10 | [Autonomous Agent](usecases/10-autonomous-agent/) | Docker + Templates |
| 11 | [Multi-Agent Swarm](usecases/11-multi-agent-swarm/) | Parallel subagents |
| 12 | [Web Research Integration](usecases/12-web-research/) | MCP (Perplexity) |

---

## Glossary

| Term | Meaning |
|------|---------|
| **CLAUDE.md** | Markdown file providing instructions to Claude Code. Auto-loaded every conversation. |
| **MCP** | Model Context Protocol - standard for connecting Claude to external tools |
| **MCP Server** | Small process exposing tools to Claude (e.g., JIRA, Slack, database) |
| **Skill** | Reusable specialist persona invoked via `/skill-name` |
| **Command** | One-shot workflow invoked via `/command-name` |
| **settings.local.json** | Personal permissions file (not committed to git) |
| **Plan Mode** | Read-only mode where Claude researches without making changes |
| **Opus / Sonnet / Haiku** | Claude model tiers: most capable / balanced / fastest |
| **Subagent** | Child Claude instance launched for a parallel subtask |
| **Bypass Permissions** | Mode where Claude runs everything without asking (use cautiously) |

---

## Resources

- [Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code)
- [MCP Server Directory](https://github.com/modelcontextprotocol/servers)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)
