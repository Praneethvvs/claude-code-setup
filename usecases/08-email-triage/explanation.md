# Explanation: Notification Triage Automation

## The General Pattern

This use case demonstrates a reusable pattern: **External API + Categorization Rules**

1. **Authenticate** to a notification source
2. **Fetch** unprocessed items
3. **Cross-reference** with another data source (e.g., JIRA assignments)
4. **Categorize** using pattern rules
5. **Act** on each category (flag, mark read, archive)
6. **Report** what was done

This same pattern works for any notification source - email, GitHub, Slack, or custom tools.

## The Categorization Logic

The pattern matching should be hierarchical:

```
1. Is it "assigned to you"?        → FLAG (highest priority)
2. Is it "mentioned you"?          → FLAG
3. Is the ticket in my assignments? → KEEP UNREAD (track updates)
4. Is it from automation?          → MARK READ (noise)
5. Everything else?                → MARK READ (FYI)
```

Priority matters. A notification about a ticket assigned to you AND from automation still gets flagged (rule 1 wins).

## Applying the Pattern to Different Sources

### GitHub Notifications (Recommended Start)

The easiest to set up because `gh` CLI is already authenticated:

```bash
# Fetch notifications
gh api notifications --jq '.[] | {subject: .subject.title, reason: .reason}'

# Mark as read
gh api -X PATCH notifications/threads/{id}

# Fetch PRs needing your review
gh pr list --search "review-requested:@me"
```

No API keys, no tokens, no MCP servers needed.

### Email (Outlook via Graph API)

For teams that get heavy JIRA/Azure DevOps email notifications:

- Uses `curl` commands against Microsoft Graph API (not an MCP server)
- OAuth token from Graph Explorer (manual, expires in ~1 hour)
- Non-destructive: flag or mark read, never delete
- Can cross-reference with JIRA MCP to check your assignments

Key consideration: token refresh is manual. For frequent use, consider automating via Azure AD app registration.

### Slack

```markdown
# /slack-triage
1. Fetch unread Slack DMs and mentions via Slack API
2. Cross-reference with JIRA assignments
3. Categorize: direct question → flag, FYI → mark read, bot notification → archive
4. Report summary
```

### PR Review Queue

```markdown
# /pr-queue
1. Fetch open PRs requesting your review: `gh pr list --search "review-requested:@me"`
2. For each PR, check: age, size, CI status
3. Prioritize: old + small + green CI → review first
4. Report ordered queue
```

## Adapting for {{ORG}}

### Start with GitHub

If {{ORG}} uses GitHub, the `/gh-triage` skill in the README requires zero external setup and gives immediate value.

### Add Email Later

If you need email triage:
- You'll need a Microsoft 365 account with Graph API access
- Token refresh is manual (copy from Graph Explorer each session)
- Consider automating token refresh via Azure AD app registration if used frequently

### Key Design Decisions

- **Non-destructive actions only**: flag or mark read, never delete or archive permanently
- **Confirmation before bulk actions**: always ask before processing more than 10-20 items
- **Summary reporting**: always show what was done so the user can verify
