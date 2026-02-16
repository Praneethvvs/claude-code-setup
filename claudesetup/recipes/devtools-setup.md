# Dev Tools Setup

> Prerequisites: [Getting Started](guide/01-getting-started.md) | Time: ~15 min

## What This Does

A `/setup` command that checks your machine for required dev tools, installs anything missing, and verifies everything works. Covers: Python, Git, Docker, Postman, VS Code, Azure CLI, AWS CLI, and Node.js.

Works on macOS (Homebrew) and Linux (apt).

## Setup

### Step 1: Create the command

```bash
mkdir -p .claude/commands
```

Create `.claude/commands/setup.md`:

```markdown
---
description: "Check and install dev tools: Python, Git, Docker, Postman, VS Code, CLIs"
allowed-tools:
  - "Bash(brew *)"
  - "Bash(apt-get *)"
  - "Bash(sudo apt-get *)"
  - "Bash(command *)"
  - "Bash(which *)"
  - "Bash(git *)"
  - "Bash(python3 *)"
  - "Bash(node *)"
  - "Bash(docker *)"
  - "Bash(az *)"
  - "Bash(aws *)"
  - "Bash(code *)"
argument-hint: "[optional: tool names to install, or 'all']"
---

# Dev Environment Setup

Check which dev tools are installed, install anything missing, and verify.

## Step 1: Detect OS and Package Manager

```bash
if [[ "$OSTYPE" == "darwin"* ]]; then
    echo "OS: macOS"
    command -v brew &>/dev/null && echo "PKG: Homebrew" || echo "PKG: MISSING brew"
elif [[ -f /etc/debian_version ]]; then
    echo "OS: Debian/Ubuntu"
    echo "PKG: apt"
else
    echo "OS: $(uname -s)"
    echo "PKG: unknown"
fi
```

If Homebrew is missing on macOS, suggest installing it first:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Step 2: Check Current State

Run all checks and report a table:

```bash
echo "=== Dev Tools Check ==="
command -v git &>/dev/null && echo "OK  git          $(git --version | head -1)" || echo "MISSING  git"
command -v python3 &>/dev/null && echo "OK  python3      $(python3 --version 2>&1)" || echo "MISSING  python3"
command -v node &>/dev/null && echo "OK  node         $(node --version)" || echo "MISSING  node"
command -v docker &>/dev/null && echo "OK  docker       $(docker --version 2>&1 | head -1)" || echo "MISSING  docker"
command -v code &>/dev/null && echo "OK  vscode       $(code --version 2>&1 | head -1)" || echo "MISSING  vscode"
command -v az &>/dev/null && echo "OK  azure-cli    $(az version --query '\"azure-cli\"' -o tsv 2>/dev/null)" || echo "MISSING  azure-cli"
command -v aws &>/dev/null && echo "OK  aws-cli      $(aws --version 2>&1 | head -1)" || echo "MISSING  aws-cli"
# Postman is a GUI app - check differently per OS
if [[ "$OSTYPE" == "darwin"* ]]; then
    [[ -d "/Applications/Postman.app" ]] && echo "OK  postman" || echo "MISSING  postman"
else
    command -v postman &>/dev/null && echo "OK  postman" || echo "MISSING  postman"
fi
```

If everything is OK, skip to Step 4.

## Step 3: Install Missing Tools

**Show the user which tools are missing and ask for confirmation before installing.**

### macOS (Homebrew)

```bash
# CLI tools
command -v git &>/dev/null || brew install git
command -v python3 &>/dev/null || brew install python@3.11
command -v node &>/dev/null || brew install node
command -v az &>/dev/null || brew install azure-cli
command -v aws &>/dev/null || brew install awscli

# GUI apps (cask)
command -v docker &>/dev/null || brew install --cask docker
command -v code &>/dev/null || brew install --cask visual-studio-code
[[ -d "/Applications/Postman.app" ]] || brew install --cask postman
```

### Linux (apt)

```bash
sudo apt-get update
command -v git &>/dev/null || sudo apt-get install -y git
command -v python3 &>/dev/null || sudo apt-get install -y python3 python3-pip python3-venv
command -v node &>/dev/null || { curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - && sudo apt-get install -y nodejs; }
command -v docker &>/dev/null || { curl -fsSL https://get.docker.com | sudo sh; }
command -v code &>/dev/null || { wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg && sudo install -o root -g root -m 644 packages.microsoft.gpg /usr/share/keyrings/ && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list && sudo apt-get update && sudo apt-get install -y code; }
command -v az &>/dev/null || { curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash; }
command -v aws &>/dev/null || { curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip" && unzip -qo /tmp/awscliv2.zip -d /tmp && sudo /tmp/aws/install; }
# Postman: snap or flatpak
command -v postman &>/dev/null || sudo snap install postman 2>/dev/null || echo "Install Postman manually: https://www.postman.com/downloads/"
```

## Step 4: Verify

Re-run the check from Step 2. Everything should show OK.

## Rules
- Always show current state before installing anything
- Ask for confirmation before installing
- Never overwrite existing installations
- If a tool can't be auto-installed, provide the manual install link

$ARGUMENTS
```

### Step 2: Add permissions

In `.claude/settings.local.json`, add to `allow`:

```json
"Bash(brew install:*)",
"Bash(brew install --cask:*)",
"Bash(brew list:*)",
"Bash(command -v:*)",
"Bash(which:*)"
```

## Try It

```
claude
> /setup
> /setup python3 docker vscode
```

Claude will check what's installed, show a summary table, and offer to install anything missing.

## Customize

**Add more tools**: Edit the command to include tools specific to your stack (e.g., `kubectl`, `terraform`, `gh`, `jq`, `redis-cli`).

**Add credential setup**: Extend the command with login steps after installation:

```markdown
## Step 5: Configure Credentials (optional)

Ask: "Do you want to configure cloud credentials now?"

- Azure: `az login && az account set --subscription "your-sub"`
- AWS: `aws configure`
- GitHub: `gh auth login`
```

**Add repo cloning**: After tools are installed, clone your team's repos:

```markdown
## Step 6: Clone Repositories (optional)

Ask: "Which repositories do you need?"

gh repo list your-org --limit 50 --json name --jq '.[].name'
```

**Make it a skill instead**: If you want multiple modes (e.g., `/setup check` vs `/setup install` vs `/setup verify`), convert it to a skill in `.claude/skills/setup/SKILL.md` using the [Skills](guide/03-skills.md) pattern.
