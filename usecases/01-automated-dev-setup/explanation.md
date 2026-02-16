# Explanation: Automated Dev Environment Setup

## How the Original Enterprise Setup Worked

A `/setup` skill can act as a 5-phase interactive wizard:

| Phase | What It Does |
|-------|-------------|
| 1. Check State | Runs diagnostic scripts to detect installed tools, missing configs, stored credentials |
| 2. Bootstrap | Clones the docs repo first (contains all config templates), copies `.claude/` config |
| 3. Collect Tokens | Walks user through creating API tokens for Azure DevOps, Atlassian, Perplexity, Anthropic |
| 4. Auto-Install | Installs Homebrew, dev tools, clones repos in parallel (8 at a time), fills MCP templates |
| 5. Verify | Runs checklist with pass/fail for every component |

### Key Design Decisions

**Credential storage**: Used macOS Keychain (`security add-generic-password`) instead of `.env` files. Credentials are never written to disk in plaintext.

**Parallel cloning**: When cloning many repos, the script used background jobs with a concurrency limit:
```bash
for repo in $REPOS; do
  [ ! -d "$repo" ] && git clone "$url" &
  while [ $(jobs -r | wc -l) -ge 8 ]; do sleep 1; done  # Max 8 parallel
done
wait
```

**Template system**: Config files use `{{PLACEHOLDER}}` variables. The setup phase fills them with `sed`:
```bash
cp templates/mcp.json.template ~/.mcp.json
sed -i '' "s/{{API_KEY}}/$ACTUAL_KEY/g" ~/.mcp.json
```

**Idempotency**: Every step checks before acting (`[ ! -d "repo" ] && git clone`). Running `/setup` twice doesn't break anything.

## Claude Features That Enable This

### Skills

Skills provide **structured multi-step workflows** that Claude follows consistently. The key advantage over just asking Claude "help me set up" is:
- Steps are defined and reproducible
- Rules are enforced (e.g., "collect all inputs first")
- The same setup works for every new team member

### Bash Permissions

Without pre-approved permissions, Claude asks for approval at every `brew install`, `git clone`, etc. With 20+ tools to install, that means 20+ approval prompts. Pre-approving install commands makes it seamless.

### Templates

Template files with placeholders allow sharing config structure without sharing secrets. The setup skill fills in placeholders with the user's actual credentials.

## Adapting for {{ORG}}

The README.md in this folder has a working {{ORG}}-specific setup skill. To extend it:

### Add More Phases

Add Azure DevOps login (separate from Azure Portal):
```bash
# Azure DevOps uses a different auth system
echo "YOUR_PAT" | az devops login --organization https://dev.azure.com/{{org}}
```

### Add Project-Specific Setup

```bash
# Create standard Python venv
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run database migrations
alembic upgrade head

# Copy .env template
cp .env.example .env
echo "Edit .env with your local settings"
```

### Add Docker Compose

```bash
# Start local services (database, cache, etc.)
docker compose up -d
docker compose ps  # Verify all services are running
```
