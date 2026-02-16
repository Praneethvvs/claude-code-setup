# Cloud Log Debugging

> Prerequisites: [Getting Started](guide/01-getting-started.md) | Time: ~15 min

## What This Does

Claude queries Azure Monitor, AWS CloudWatch, and Kubernetes logs directly from your terminal. You say "the staging API is returning 500s" and Claude finds the error, explains the root cause, and suggests a fix.

## Setup

### Step 1: Add cloud context to CLAUDE.md

Claude needs to know your resource names, IDs, and how to query them. Add a section like this to your project's `CLAUDE.md`:

```markdown
## Cloud Resources

### Azure
| Resource | Name | Resource Group |
|----------|------|---------------|
| App Service (API) | myorg-api-staging | rg-myorg-staging |
| App Insights | ai-myorg-staging | rg-myorg-staging |
| Log Analytics | la-myorg-staging | rg-myorg-staging |

### AWS
| Resource | ID/Name | Region |
|----------|---------|--------|
| ECS Cluster | myorg-staging | us-east-1 |
| CloudWatch Group | /ecs/myorg-api | us-east-1 |
| Lambda | myorg-data-processor | us-east-1 |

### Debug Commands Quick Reference
- Azure App Insights: `az monitor app-insights query --app ai-myorg-staging --analytics-query "requests | where success == false | top 10 by timestamp desc"`
- AWS CloudWatch: `aws logs filter-log-events --log-group-name /ecs/myorg-api --filter-pattern "ERROR"`
- K8s pod logs: `kubectl logs -n staging deployment/myorg-api --tail=100`
```

Replace all resource names with your actual values.

### Step 2: Add permissions

In `.claude/settings.local.json`, add to `allow`:

```json
"Bash(az account show:*)",
"Bash(az account set:*)",
"Bash(az monitor app-insights query:*)",
"Bash(az monitor log-analytics query:*)",
"Bash(az webapp config appsettings list:*)",
"Bash(az webapp show:*)",
"Bash(aws logs filter-log-events:*)",
"Bash(aws logs describe-log-groups:*)",
"Bash(aws ecs describe-tasks:*)",
"Bash(aws ecs describe-services:*)",
"Bash(kubectl logs:*)",
"Bash(kubectl get:*)",
"Bash(kubectl describe:*)"
```

### Step 3: Verify CLI access

Make sure your CLIs are authenticated before using Claude:

```bash
az login          # Azure
aws sts get-caller-identity  # AWS
kubectl get pods  # Kubernetes
```

## Try It

```
claude
> The staging API is returning 500 errors, can you check the logs?
> What errors happened in the last hour on the ECS cluster?
> Check if the Lambda function myorg-data-processor has any recent failures
```

Claude will pick the right CLI tool, construct the query, and explain what it finds.

## Customize

**Add more resource types**: Extend the table in CLAUDE.md with databases, queues, storage accounts, etc.

**Add environment switching**: Include a table of environments so Claude can query the right one:

```markdown
## Environments
| Env | Azure Subscription | AWS Profile |
|-----|-------------------|-------------|
| Dev | myorg-dev | dev |
| Staging | myorg-staging | staging |
| Prod | myorg-prod | prod |
```

**Deny destructive commands**: Make sure you're not allowing write operations on cloud resources:

```json
"deny": [
  "Bash(az webapp delete:*)",
  "Bash(az webapp restart:*)",
  "Bash(aws ecs update-service:*)"
]
```
