# Explanation: Debugging Cloud Deployment Logs

## How the Original Enterprise Setup Worked

A production enterprise setup had a dual-cloud environment (Azure primary, AWS secondary) with nuances like **workspace-based App Insights** vs classic App Insights, which changes which CLI command to use (`az monitor log-analytics query` vs `az monitor app-insights query`).

They also had two separate Azure auth systems:
- `az login` for Portal resources (VMs, App Services)
- `az devops login` for DevOps (repos, PRs, pipelines)

All of this was documented in CLAUDE.md with specific workspace IDs, environment mappings, and the correct CLI commands. Without this context, Claude would use the wrong command every time.

### The "Required Reading" Pattern

For complex operations, CLAUDE.md pointed to detailed guides:

```markdown
| Task | Read First |
|------|-----------|
| Azure CLI commands | docs/claude-assistant/operations/azure_guide.md |
| Troubleshooting errors | docs/claude-assistant/operations/troubleshooting.md |
```

These detailed guides contained KQL query examples, common error patterns, and step-by-step troubleshooting flows. This is more context than fits in CLAUDE.md, but Claude reads them on-demand before acting.

## Claude Features That Enable This

### CLAUDE.md (Critical Context)

The key insight: Claude can run any CLI command, but it needs to know **which resources exist and how to identify them**. Without resource names, subscription IDs, and workspace IDs, Claude is guessing.

The pattern is:
1. Document your resources in CLAUDE.md (names, IDs, resource groups)
2. Document the correct CLI commands for your specific setup
3. Let Claude combine the context with its knowledge of the CLI

### Bash Permissions (Read-Only Operations)

The permission list is carefully curated: **all read-only operations are pre-approved**. Write operations (restart, deploy, config changes) require manual approval.

This means Claude can freely investigate problems but must ask before making changes. This is the safest pattern for production debugging.

### The Required Reading Pattern

For deep operational context that doesn't fit in CLAUDE.md:
1. Create detailed guides in your `docs/` folder
2. Reference them in CLAUDE.md's "Required Reading" table
3. Claude reads the full guide before performing the task

This keeps CLAUDE.md lean while giving Claude access to unlimited depth.

## Adapting for {{ORG}}

### Multi-Cloud Strategy

Since {{ORG}} uses both Azure and AWS, add a routing table to CLAUDE.md:

```markdown
## Service-to-Cloud Mapping

| Service | Cloud | How to Debug |
|---------|-------|-------------|
| API Gateway | Azure App Service | `az monitor app-insights query` |
| Data Pipeline | AWS Lambda + ECS | `aws logs filter-log-events` |
| Frontend | Azure Static Web Apps | `az staticwebapp` |
| ML Models | AWS SageMaker | `aws sagemaker` |
| Database | Azure SQL | `az sql` |
| File Storage | AWS S3 | `aws s3` |
```

### Common Debug Scenarios

Add a troubleshooting section to CLAUDE.md:

```markdown
## Common Issues

| Symptom | Where to Look | Command |
|---------|--------------|---------|
| API 500 errors | Azure App Insights | `az monitor app-insights query --app ai-{{org}}-staging --analytics-query "requests | where resultCode == 500"` |
| Lambda timeouts | AWS CloudWatch | `aws logs filter-log-events --log-group /aws/lambda/{{org}}-data-processor --filter-pattern "Task timed out"` |
| Pod crashes | Kubernetes events | `kubectl get events -n staging --field-selector reason=BackOff` |
| Missing env var | App Service config | `az webapp config appsettings list --name {{org}}-api-staging -g rg-{{org}}-staging` |
```

### Create a Troubleshooting Guide

For common patterns, create `docs/troubleshooting.md` and reference it:

```markdown
## Required Reading
| Task | Read First |
|------|-----------|
| Debugging any error | docs/troubleshooting.md |
```

The guide can contain KQL templates, CloudWatch Insights queries, and decision trees for common error patterns.
