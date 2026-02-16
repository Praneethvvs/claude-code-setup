# 01: Automated Dev Environment Setup

## What It Does

Claude walks new {{ORG}} developers through setting up their entire environment interactively:
- Checks what's already installed
- Installs missing tools (Python, Docker, Azure CLI, AWS CLI, git, etc.)
- Clones repositories
- Configures credentials securely
- Verifies everything works

## Why It Matters

New developers spend hours (or days) setting up. This reduces it to a single conversation: "Help me set up."

## Prerequisites

- macOS with Homebrew (or Linux with apt)
- Claude Code installed (`npm install -g @anthropic-ai/claude-code`)

## Setup Steps

### 1. Create the skill file

```bash
mkdir -p .claude/skills/setup
```

Create `.claude/skills/setup/SKILL.md`:

```markdown
---
name: setup
description: {{ORG}} dev environment setup wizard
---

# /setup - {{ORG}} Environment Setup

**Purpose**: Interactive setup wizard for new {{ORG}} developers.

## WORKFLOW

### Step 1: Check Current State

```bash
echo "=== {{ORG}} Environment Check ==="
command -v git &>/dev/null && echo "OK git" || echo "MISSING git"
command -v python3 &>/dev/null && echo "OK python3 ($(python3 --version 2>&1))" || echo "MISSING python3"
command -v docker &>/dev/null && echo "OK docker" || echo "MISSING docker"
command -v az &>/dev/null && echo "OK azure-cli" || echo "MISSING azure-cli"
command -v aws &>/dev/null && echo "OK aws-cli" || echo "MISSING aws-cli"
command -v gh &>/dev/null && echo "OK github-cli" || echo "MISSING github-cli"
command -v jq &>/dev/null && echo "OK jq" || echo "MISSING jq"
command -v node &>/dev/null && echo "OK node" || echo "MISSING node"
```

If nothing missing, skip to Step 4 (verify).

### Step 2: Install Missing Tools

```bash
command -v git &>/dev/null || brew install git
command -v python3 &>/dev/null || brew install python@3.11
command -v docker &>/dev/null || brew install --cask docker
command -v az &>/dev/null || brew install azure-cli
command -v aws &>/dev/null || brew install awscli
command -v gh &>/dev/null || brew install gh
command -v jq &>/dev/null || brew install jq
command -v node &>/dev/null || brew install node
```

### Step 3: Configure Credentials

Say: "Let's configure your cloud credentials."

#### Azure
```bash
az login
az account set --subscription "{{org}}-dev"
```

#### AWS
```bash
aws configure
# Prompt for: Access Key, Secret Key, Region (us-east-1), Output (json)
```

#### GitHub
```bash
gh auth login
```

### Step 4: Clone Repositories

Ask: "Which repositories do you need? Or should I clone the core set?"

```bash
gh repo list {{org}} --limit 50 --json name,url | jq -r '.[].name'
```

### Step 5: Python Environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Step 6: Verify

```bash
echo "=== Final Verification ==="
git --version && echo "OK git"
python3 --version && echo "OK python3"
docker --version && echo "OK docker"
az version --output table && echo "OK azure-cli"
aws --version && echo "OK aws-cli"
gh auth status && echo "OK github"
echo "=== Setup Complete ==="
```

## RULES
1. Collect all inputs before running automated steps
2. Explain each step in plain English
3. Ask before destructive or overwriting actions
4. Store credentials securely (never plaintext files)
```

### 2. Allow required permissions

Add to `.claude/settings.local.json` `allow` list:

```json
"Bash(brew install:*)",
"Bash(brew list:*)",
"Bash(az login:*)",
"Bash(az account set:*)",
"Bash(aws configure:*)",
"Bash(gh auth login:*)",
"Bash(gh repo list:*)",
"Bash(pip install:*)"
```

### 3. Use it

```
claude
> /setup
```

## See Also

- [explanation.md](explanation.md) - Deep dive into how the original enterprise setup worked
