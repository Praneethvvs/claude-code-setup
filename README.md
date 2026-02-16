# Claude Code Setup Guide

A practical guide to setting up Claude Code for any project. Learn the concepts progressively, then pick standalone recipes for specific workflows.

---

## How to Use This Guide

**Read the Guide in order** (6 pages, ~90 min total). Each page teaches one concept and has you build one thing. By the end, you'll have a fully configured Claude Code setup.

**Pick Recipes as needed.** Each recipe is standalone and solves a specific problem.

---

## Guide

Read these in order. Each builds on the previous.

| # | Page | You'll Learn | You'll Build |
|---|------|-------------|-------------|
| 1 | [Getting Started](guide/01-getting-started.md) | CLAUDE.md, permissions, .claudeignore | A working project setup |
| 2 | [Commands](guide/02-commands.md) | How commands work | A /compose command for smart git commits |
| 3 | [Skills](guide/03-skills.md) | How skills work | A Python code expert skill |
| 4 | [MCP Servers](guide/04-mcp-servers.md) | How MCP connects external tools | Context7 + WebSearch setup |
| 5 | [Automation](guide/05-automation.md) | Headless mode + CI/CD | A template system + GitHub Actions review |
| 6 | [Parallel Agents](guide/06-parallel-agents.md) | Multi-agent parallelism | A shell-based parallel runner |

---

## Recipes

Standalone. Pick what you need.

| Recipe | What It Does | Key Feature |
|--------|-------------|-------------|
| [Dev Tools Setup](recipes/devtools-setup.md) | Install Python, Git, Docker, Postman, VS Code, CLIs | Command + brew/apt |
| [Cloud Debugging](recipes/cloud-debugging.md) | Query Azure/AWS/K8s logs from terminal | CLI permissions + CLAUDE.md context |
| [Story Estimation](recipes/story-estimation.md) | Fibonacci complexity scoring from codebase analysis | Skill with scoring matrix |
| [GitHub Triage](recipes/github-triage.md) | Prioritize notifications, clear noise | Skill + `gh` CLI |
| [Self-Improving](recipes/self-improving.md) | `/reflect` to improve CLAUDE.md over time | Feedback loop skill |

---

## Quick Reference

See the [Reference](reference.md) page for: glossary, permissions cheatsheet, model IDs, and common errors.
