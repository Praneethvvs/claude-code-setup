# Reading Plan

Work top to bottom. Each step builds on the previous one.

---

## Step 1: Core Concepts (20 min read, then do)

**Read** `claudesetup/README.md` sections 1-4 (stop at "Permissions")

You're learning: what the 6 config files are, and how CLAUDE.md works.

**Do**: Create a CLAUDE.md in any project and run `claude`. Have a conversation. That's it.

- [ ] Done

---

## Step 2: Permissions (10 min read, then do)

**Read** `claudesetup/README.md` section 5 (Permissions)

You're learning: how to pre-approve safe commands and block dangerous ones.

**Do**: Create `.claude/settings.local.json` with the starter permissions from section 2 (Quick Start). Run `claude` again - notice it no longer asks permission for git/python commands.

- [ ] Done

---

## Step 3: Your First Command (10 min)

**Read** `usecases/06-smart-git-commits/README.md`

You're learning: how commands work (one file = one workflow).

**Do**: Create `.claude/commands/compose.md`, make some code changes, then type `/compose` in Claude.

- [ ] Done

---

## Step 4: MCP Servers (15 min)

**Read** `claudesetup/README.md` section 6 (MCP Servers)

You're learning: how to connect Claude to external services.

**Do**: Pick ONE server that's useful to you right now (Context7 is the easiest - no API key needed). Add it to `.mcp.json`, restart Claude.

- [ ] Done

**Troubleshooting**: MCP servers only load on startup. If you add one, you must `exit` and run `claude` again.

---

## Step 5: Your First Skill (20 min)

**Read** `claudesetup/README.md` sections 7-8 (Skills + Commands)

Then **read** `usecases/07-code-expert-pr-review/README.md`

You're learning: skills = specialist personas with rules, workflows, and model selection.

**Do**: Create `.claude/skills/python/SKILL.md` - start with just the ALWAYS/NEVER rules from your own codebase. Invoke it with `/python review` on a file.

- [ ] Done

---

## Step 6: Best Practices + Reflect (10 min)

**Read** `claudesetup/README.md` section 9 (Best Practices)

Then **read** `usecases/09-self-improving-config/README.md`

**Do**: At the end of your next real work session, ask Claude: "What should we add to CLAUDE.md based on this session?"

- [ ] Done

---

## Step 7: Browse Use Cases (when needed)

You now know the 4 building blocks: CLAUDE.md, permissions, commands, skills.

The remaining use cases in `usecases/` are applications of those blocks. **Don't read them all now.** Open `usecases/README.md` to see the index, then read individual ones when you have a matching problem:

| Problem | Read |
|---------|------|
| Need to debug cloud logs | `02-debugging-cloud-logs/` |
| Want JIRA/Confluence integration | `03-jira-confluence-automation/` |
| Want to automate specs to tickets | `04-product-spec-pipeline/` |
| Want story point estimation | `05-story-point-estimation/` |
| Want dev environment automation | `01-automated-dev-setup/` |
| Want notification triage (GitHub/email/Slack) | `08-email-triage/` |
| Want headless/CI Claude | `10-autonomous-agent/` |
| Want parallel agents | `11-multi-agent-swarm/` |
| Want better web research | `12-web-research/` |

Each folder has a README.md (setup steps) and explanation.md (how it works).

- [ ] Browsed the index

---

## Glossary (reference - don't memorize)

| Term | What it is |
|------|-----------|
| **CLAUDE.md** | Instructions file, auto-loaded every session |
| **MCP Server** | Plugin that gives Claude access to an external service |
| **Skill** | Specialist persona invoked with `/skill-name` |
| **Command** | One-shot workflow invoked with `/command-name` |
| **Opus/Sonnet/Haiku** | Model tiers: most capable / balanced / fastest |

---

## Progress

| Step | Status |
|------|--------|
| 1. Core Concepts | |
| 2. Permissions | |
| 3. First Command | |
| 4. MCP Servers | |
| 5. First Skill | |
| 6. Best Practices | |
| 7. Browse Use Cases | |
