# 03: JIRA/Confluence Automation

## What It Does

Claude reads, searches, and creates JIRA tickets and Confluence pages directly from your terminal. No browser needed for:
- "Show me the current sprint"
- "What does {{ORG}}-123 involve?" (reads ticket + all linked tickets + Confluence docs)
- "Create a ticket for the auth bug we discussed"
- "Search Confluence for the API docs"

## Why It Matters

Context switching between terminal and JIRA/Confluence is a constant productivity drain. This keeps you in flow.

## Prerequisites

- JIRA/Confluence instance (e.g., `{{org}}.atlassian.net`)
- Atlassian API token
- Docker installed (for MCP server)

## Setup Steps

### 1. Get an Atlassian API Token

1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click **Create API token**
3. Name: `Claude Code`
4. Copy the token

### 2. Create `.mcp.json` at project root

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-e", "CONFLUENCE_URL",
        "-e", "CONFLUENCE_USERNAME",
        "-e", "CONFLUENCE_API_TOKEN",
        "-e", "JIRA_URL",
        "-e", "JIRA_USERNAME",
        "-e", "JIRA_API_TOKEN",
        "ghcr.io/sooperset/mcp-atlassian:latest"
      ],
      "env": {
        "CONFLUENCE_URL": "https://{{org}}.atlassian.net/wiki",
        "CONFLUENCE_USERNAME": "you@{{org}}.com",
        "CONFLUENCE_API_TOKEN": "YOUR_TOKEN_HERE",
        "JIRA_URL": "https://{{org}}.atlassian.net",
        "JIRA_USERNAME": "you@{{org}}.com",
        "JIRA_API_TOKEN": "YOUR_TOKEN_HERE"
      }
    }
  }
}
```

**Add `.mcp.json` to `.gitignore`** (contains your token).

### 3. Add JIRA context to CLAUDE.md

```markdown
## JIRA

- Organization: https://{{org}}.atlassian.net
- Project key: {{ORG}} (adjust to your project)
- Board ID: [find at your-org.atlassian.net/jira/software/projects/{{ORG}}/boards]
- Sprint field: customfield_10015

### Ticket Rules
- Always show ticket details for approval before creating
- Always read ALL linked tickets before answering about a ticket
- Link child tickets to parent Epic using Parent-Child (not Subtask)
- Include acceptance criteria in every Story
- Priority: Medium unless specified
```

### 4. Add permissions

In `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__mcp-atlassian__jira_search",
      "mcp__mcp-atlassian__jira_get_issue",
      "mcp__mcp-atlassian__jira_get_sprint_issues",
      "mcp__mcp-atlassian__jira_get_agile_boards",
      "mcp__mcp-atlassian__jira_get_sprints_from_board",
      "mcp__mcp-atlassian__confluence_search"
    ]
  },
  "enableAllProjectMcpServers": true
}
```

Note: Write operations (create/update) are NOT pre-approved. Claude will ask first.

### 5. Restart Claude Code

MCP servers load at startup:
```bash
exit
claude
```

### 6. Use it

```
> Show me the current sprint
> What does {{ORG}}-42 involve?
> Create a story for adding user export functionality
```

## See Also

- [explanation.md](explanation.md) - How the Atlassian MCP server works, available tools, JQL tips
