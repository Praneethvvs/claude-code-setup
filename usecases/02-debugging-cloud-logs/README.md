# 02: Debugging Cloud Deployment Logs

## What It Does

Claude directly queries Azure Monitor, AWS CloudWatch, and Kubernetes logs from your terminal. It finds the error, explains the root cause, and suggests a fix - no browser tab switching.

## Why It Matters

Debugging cloud deployments requires knowing the right CLI tool, the right query syntax, and the right resource IDs. Claude handles all of that if you give it the context.

## Prerequisites

- Azure CLI logged in (`az login`)
- AWS CLI configured (`aws configure`)
- kubectl configured (if using K8s)

## Setup Steps

### 1. Add cloud context to CLAUDE.md

```markdown
## Cloud Resources

### Azure
| Resource | Name | Resource Group | Subscription |
|----------|------|---------------|-------------|
| App Service (API) | {{org}}-api-staging | rg-{{org}}-staging | {{org}}-dev |
| App Service (Web) | {{org}}-web-staging | rg-{{org}}-staging | {{org}}-dev |
| Key Vault | kv-{{org}}-staging | rg-{{org}}-staging | {{org}}-dev |
| App Insights | ai-{{org}}-staging | rg-{{org}}-staging | {{org}}-dev |
| Log Analytics | la-{{org}}-staging | rg-{{org}}-staging | {{org}}-dev |

### AWS
| Resource | ID/Name | Region |
|----------|---------|--------|
| ECS Cluster | {{org}}-staging | us-east-1 |
| CloudWatch Group | /ecs/{{org}}-api | us-east-1 |
| S3 Bucket | {{org}}-staging-assets | us-east-1 |
| Lambda | {{org}}-data-processor | us-east-1 |

### Kubernetes (if applicable)
| Namespace | Environment | Cluster |
|-----------|------------|---------|
| staging | Pre-prod | {{org}}-aks-staging |
| production | Live | {{org}}-aks-prod |

## Debug Commands Quick Reference

### Azure Logs
- App Insights: `az monitor app-insights query --app ai-{{org}}-staging --analytics-query "requests | where success == false | top 10 by timestamp desc"`
- Log Analytics: `az monitor log-analytics query --workspace la-{{org}}-staging --analytics-query "AppExceptions | top 10 by TimeGenerated desc"`
- App config: `az webapp config appsettings list --name {{org}}-api-staging -g rg-{{org}}-staging`

### AWS Logs
- CloudWatch: `aws logs filter-log-events --log-group-name /ecs/{{org}}-api --filter-pattern "ERROR" --start-time $(date -d '1 hour ago' +%s000)`
- ECS tasks: `aws ecs describe-tasks --cluster {{org}}-staging --tasks <task-id>`
- Lambda logs: `aws logs filter-log-events --log-group-name /aws/lambda/{{org}}-data-processor --filter-pattern "ERROR"`

### Kubernetes
- Pod logs: `kubectl logs -n staging deployment/{{org}}-api --tail=100`
- Events: `kubectl get events -n staging --sort-by='.lastTimestamp'`
- Pod status: `kubectl get pods -n staging`
```

### 2. Add permissions

In `.claude/settings.local.json`, add to `allow`:

```json
"Bash(az monitor app-insights query:*)",
"Bash(az monitor log-analytics query:*)",
"Bash(az webapp config appsettings list:*)",
"Bash(az keyvault secret show:*)",
"Bash(aws logs filter-log-events:*)",
"Bash(aws logs describe-log-groups:*)",
"Bash(aws ecs describe-tasks:*)",
"Bash(aws ecs describe-services:*)",
"Bash(kubectl logs:*)",
"Bash(kubectl get:*)",
"Bash(kubectl describe:*)"
```

### 3. Use it

```
claude
> The staging API is returning 500 errors, can you check the logs?
```

Claude will query the right service, find the errors, and explain what's wrong.

## See Also

- [explanation.md](explanation.md) - How log debugging works under the hood
