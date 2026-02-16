# Claude Code Use Cases for {{ORG}}

Each folder below documents a real-world problem Claude Code can solve, which Claude features enable it, and how to set it up for {{ORG}} projects.

## Use Case Index

| # | Use Case | Complexity | Key Feature | Setup Time |
|---|----------|-----------|-------------|------------|
| 01 | [Automated Dev Environment Setup](01-automated-dev-setup/) | Medium | Skills + Bash | 1-2 hours |
| 02 | [Debugging Cloud Deployment Logs](02-debugging-cloud-logs/) | Easy | CLAUDE.md + Permissions | 15 min |
| 03 | [JIRA/Confluence Automation](03-jira-confluence-automation/) | Medium | MCP Server (Atlassian) | 30 min |
| 04 | [Product Spec to JIRA Pipeline](04-product-spec-pipeline/) | Medium | Commands | 1 hour |
| 05 | [AI Story Point Estimation](05-story-point-estimation/) | Hard | Skills + MCP | 2 hours |
| 06 | [Smart Git Commit Composition](06-smart-git-commits/) | Easy | Commands | 10 min |
| 07 | [Code Expert / PR Review](07-code-expert-pr-review/) | Medium | Skills | 1-2 hours |
| 08 | [Notification Triage Automation](08-email-triage/) | Medium | Skills + APIs | 30 min |
| 09 | [Self-Improving Config](09-self-improving-config/) | Easy | Skills (reflect) | 15 min |
| 10 | [Autonomous Agent](10-autonomous-agent/) | Hard | Docker + Templates | 3+ hours |
| 11 | [Multi-Agent Swarm](11-multi-agent-swarm/) | Hard | Parallel subagents | 3+ hours |
| 12 | [Web Research Integration](12-web-research/) | Easy | MCP (Perplexity) | 15 min |

## Recommended Starting Order

If you're new to Claude Code, set these up in this order:

1. **Start here** - [02: Cloud Log Debugging](02-debugging-cloud-logs/) - just CLAUDE.md + permissions, immediate value
2. **Quick win** - [06: Smart Git Commits](06-smart-git-commits/) - one command file, saves time daily
3. **Quick win** - [12: Web Research](12-web-research/) - one MCP server, no code
4. **Quick win** - [09: Self-Improving Config](09-self-improving-config/) - makes everything else better over time
5. **High value** - [03: JIRA/Confluence](03-jira-confluence-automation/) - if you use Atlassian
6. **High value** - [07: Code Expert](07-code-expert-pr-review/) - if you want consistent reviews
7. **High value** - [01: Dev Setup](01-automated-dev-setup/) - if onboarding is painful
8. **Advanced** - Everything else as needs arise

## Folder Structure

Each use case folder contains:

```
NN-use-case-name/
├── README.md          # Quick overview: what, why, how to set up (5 min read)
└── explanation.md     # Deep dive: how it works, full examples, customization
```

- **README.md** = "I want to set this up now" (setup steps, copy-paste configs)
- **explanation.md** = "I want to understand how this works" (architecture, examples, theory)
