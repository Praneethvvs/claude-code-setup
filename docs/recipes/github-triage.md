# GitHub Notification Triage

> Prerequisites: [Skills](guide/03-skills.md) | Time: ~15 min

## What This Does

A skill that fetches your GitHub notifications and PR review requests, categorizes them by priority, and marks low-priority ones as read. Turns 30+ daily notifications into a short, prioritized list.

## Setup

### Step 1: Make sure `gh` CLI is authenticated

```bash
gh auth status
```

If not authenticated: `gh auth login`

### Step 2: Create the skill

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
gh api notifications --jq '.[] | {id, subject: .subject.title, type: .subject.type, reason: .reason, updated: .updated_at, url: .subject.url}'
```

### Step 2: Fetch PR Review Requests
```bash
gh pr list --search "review-requested:@me" --json number,title,createdAt,additions,deletions,url
```

### Step 3: Categorize

| Reason | Priority | Action |
|--------|----------|--------|
| review_requested | HIGH | Show with PR details |
| assign | HIGH | Show with link |
| mention | MEDIUM | Show with context |
| ci_activity (failed) | MEDIUM | Show failure details |
| ci_activity (passed) | LOW | Mark as read |
| subscribed (no mention) | LOW | Mark as read |

### Step 4: Mark Low-Priority as Read
For each LOW priority notification:
```bash
gh api --method PATCH notifications/threads/{id}
```

### Step 5: Report

## GitHub Triage Summary

### Action Required (HIGH)
| # | Type | Title | Why |
|---|------|-------|-----|
| 1 | PR Review | [title] | Review requested |

### Worth Reading (MEDIUM)
| # | Type | Title | Why |
|---|------|-------|-----|
| 1 | Issue | [title] | You were mentioned |

### Cleared (LOW)
Marked X notifications as read (CI passes, subscribed updates).

## RULES
1. Never dismiss or unsubscribe - only mark as read
2. Always show a summary of actions taken
3. Confirm before processing if > 20 notifications
```

### Step 3: Add permissions

In `.claude/settings.local.json`, add to `allow`:

```json
"Bash(gh api:*)",
"Bash(gh pr:*)",
"Bash(gh issue:*)"
```

## Try It

```
claude
> /gh-triage
```

Claude fetches your notifications, categorizes them, marks the noise as read, and gives you a prioritized list.

## Customize

**Add repo filtering**: If you only care about certain repos, modify the fetch step:

```bash
gh api notifications --jq '.[] | select(.repository.full_name | startswith("myorg/"))'
```

**Adjust priority rules**: Add more patterns to the categorization table. For example, mark Dependabot PRs as LOW:

```markdown
| author is dependabot | LOW | Mark as read |
```

**Schedule it**: Run it every morning with a cron job:

```bash
# Add to crontab: crontab -e
0 9 * * 1-5 echo "/gh-triage" | claude --print > ~/gh-triage-$(date +\%Y-\%m-\%d).md
```
