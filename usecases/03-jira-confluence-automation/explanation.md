# Explanation: JIRA/Confluence Automation

## How the Original Enterprise Setup Worked

A well-configured setup will have comprehensive Atlassian integration with 15+ pre-approved MCP tools covering reads, searches, and writes. Key rules in CLAUDE.md ensure safety:

- **Always show for approval before creating tickets** - Claude drafts the ticket in chat, you approve, then it creates
- **Always read ALL linked tickets** - before answering about a ticket, Claude reads the parent epic, child tasks, blocking issues, and mentioned tickets
- **Never use Subtask type** - use Task with Parent-Child link (Atlassian best practice)
- **Always read the integration guide first** - a detailed doc with field mappings, JQL patterns, and edge cases

They also had a dedicated "Required Reading" file (`atlassian_mcp.md`) that Claude had to read before any JIRA/Confluence operation. This contained:
- Custom field IDs (story points = `customfield_10021`, sprint = `customfield_10015`)
- JQL query patterns for common searches
- Ticket structure conventions (prefer Story + Tasks over Epic + Stories)
- Board IDs and sprint assignment procedures

## Available MCP Tools

The Atlassian MCP server exposes these tools:

### JIRA Read Operations
| Tool | What It Does |
|------|-------------|
| `jira_get_issue` | Read full ticket details (description, status, links, custom fields) |
| `jira_search` | Search tickets with JQL queries |
| `jira_get_agile_boards` | List all boards |
| `jira_get_sprints_from_board` | List sprints for a board |
| `jira_get_sprint_issues` | Get all tickets in a sprint |
| `jira_get_board_issues` | Get all tickets on a board |
| `jira_get_transitions` | Get available status transitions |
| `jira_get_issue_link_types` | Get link types (blocks, relates to, etc.) |
| `jira_batch_get_changelogs` | Get change history for tickets |
| `jira_get_worklog` | Get time tracking entries |

### JIRA Write Operations
| Tool | What It Does |
|------|-------------|
| `jira_create_issue` | Create a new ticket |
| `jira_update_issue` | Update existing ticket fields |
| `jira_transition_issue` | Move ticket to different status |
| `jira_add_comment` | Add comment to ticket |
| `jira_link_issues` | Create links between tickets |

### Confluence Operations
| Tool | What It Does |
|------|-------------|
| `confluence_search` | Search pages by title/content |
| `confluence_get_page` | Read full page content |
| `confluence_create_page` | Create new page |
| `confluence_update_page` | Update existing page |

## JQL Quick Reference

JQL (JIRA Query Language) is how Claude searches tickets:

```
# My open tickets
assignee = currentUser() AND status != Done

# Current sprint tickets
Sprint in openSprints() AND project = {{ORG}}

# Recent bugs
project = {{ORG}} AND type = Bug AND created >= -7d ORDER BY created DESC

# Unestimated stories
project = {{ORG}} AND type = Story AND "Story Points" is EMPTY

# Tickets with keyword
project = {{ORG}} AND (summary ~ "auth" OR description ~ "auth")
```

## Adapting for {{ORG}}

### Custom Fields

Every Atlassian instance has different custom field IDs. Find yours:

```
> Search JIRA for field IDs using: jira_search_fields with query "story points"
```

Or use the Atlassian REST API:
```bash
curl -u you@{{org}}.com:YOUR_TOKEN https://{{org}}.atlassian.net/rest/api/3/field | jq '.[] | select(.name | test("story|sprint|point"; "i")) | {id, name}'
```

Add the results to CLAUDE.md:
```markdown
## JIRA Custom Fields
| Field | ID | Notes |
|-------|-----|-------|
| Story Points | customfield_10021 | Fibonacci values |
| Sprint | customfield_10015 | Board assignment |
```

### Team-Specific Rules

Add rules that match your team's workflow:

```markdown
## JIRA Conventions
- Ticket types: Epic > Story > Task (never Subtask)
- Story format: "As a [role], I want [thing], so that [benefit]"
- Every Story must have Acceptance Criteria
- Tasks under a Story should be independently assignable
- Bugs always get Priority (Critical/High/Medium/Low)
- Link related tickets with "relates to"
- Sprint capacity: ~40 story points per developer
```

### Confluence Page Templates

If you publish specs to Confluence, add structure conventions:

```markdown
## Confluence Conventions
- Space: {{ORG}} (all product docs)
- Page hierarchy: Project > Feature > Specs
- Labels: product-spec, technical-spec, runbook
- Use Markdown format (not wiki markup)
```
