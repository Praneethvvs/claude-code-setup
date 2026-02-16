# 08: Notification Triage Automation

## What It Does

Claude connects to your notification sources (email, GitHub, Slack) and automatically sorts them:
- **Flags** notifications where you're assigned or mentioned
- **Marks as read** FYI notifications and automated alerts
- **Keeps unread** updates on your active tickets

## Why It Matters

Developers on active projects get 30-50+ notifications daily. Manually sorting them wastes 15-20 minutes. This does it in seconds.

## Prerequisites

Depends on which source you triage:
- **GitHub**: `gh` CLI (already available)
- **Email (Outlook)**: Microsoft Graph API OAuth token
- **Slack**: Slack API token
- **JIRA emails**: JIRA MCP server (for checking your assignments)

## Setup Steps

### Option A: GitHub Notification Triage (Easiest)

```bash
mkdir -p .claude/skills/gh-triage
```

Create `.claude/skills/gh-triage/SKILL.md`:

```markdown
---
name: gh-triage
description: Triage GitHub notifications and PR review requests
---

# /gh-triage - GitHub Notification Triage

## WORKFLOW

### Step 1: Fetch Notifications
```bash
gh api notifications --jq '.[] | {id, subject: .subject.title, type: .subject.type, reason: .reason, updated: .updated_at}'
```

### Step 2: Fetch PR Review Requests
```bash
gh pr list --search "review-requested:@me" --json number,title,createdAt,additions,deletions
```

### Step 3: Categorize
| Reason | Priority | Action |
|--------|----------|--------|
| review_requested | HIGH | Show with PR details |
| assign | HIGH | Show with link |
| mention | MEDIUM | Show with context |
| ci_activity (failed) | MEDIUM | Show failure details |
| ci_activity (passed) | LOW | Mark as read |
| other | LOW | Mark as read |

### Step 4: Report
Show prioritized list grouped by priority, with links.

## RULES
1. Never dismiss notifications - only mark as read
2. Always show summary of actions taken
3. Confirm before processing >20 notifications
```

### Option B: Email Triage (Outlook / Graph API)

```bash
mkdir -p .claude/skills/email-triage
```

Create `.claude/skills/email-triage/SKILL.md`:

```markdown
---
name: email-triage
description: Triage notification emails - flag important, mark FYI as read
---

# /email-triage - Email Triage

## Prerequisites
- OAuth token with Mail.ReadWrite scope (get from Graph Explorer)

## WORKFLOW

### Step 1: Get Token
Ask user for a fresh token if not provided.
Direct them to: https://developer.microsoft.com/en-us/graph/graph-explorer

### Step 2: Get My Assignments (if using JIRA)
Use jira_search:
  jql: "assignee = currentUser() AND status != Done"
Extract ticket keys into a list.

### Step 3: Fetch Unread Notification Emails
```bash
TOKEN="<token>"
curl -s -H "Authorization: Bearer $TOKEN" \
  'https://graph.microsoft.com/v1.0/me/messages?$filter=isRead eq false&$select=id,subject,from,receivedDateTime&$top=50'
```

### Step 4: Categorize Each Email
| Pattern | Action |
|---------|--------|
| "assigned to you" | Flag |
| "mentioned you" | Flag |
| Ticket in my assignments | Keep unread |
| From automation | Mark read |
| Other notifications | Mark read |

### Step 5: Report Summary
Show what was flagged, marked read, and kept unread.

## RULES
1. Never delete emails - only mark read or flag
2. Always show summary of actions taken
3. Confirm before processing >10 emails
4. If token expired (401), prompt user for fresh token
```

### Add permissions

```json
"Bash(curl:*)",
"Bash(gh api:*)"
```

### Use it

```
> /gh-triage
> /email-triage
```

## See Also

- [explanation.md](explanation.md) - The general triage pattern, adapting for Slack and other sources
