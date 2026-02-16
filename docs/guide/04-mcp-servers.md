# MCP Servers

> Time: ~15 min | You will learn: **how MCP servers connect Claude to external tools** | You will build: **a setup with Context7 + WebSearch**

## What You're Building

An MCP configuration that gives Claude access to up-to-date library documentation (Context7) and web search, so it can answer questions about current APIs instead of relying on training data.

## The Concept

MCP (Model Context Protocol) servers are small processes that expose tools Claude can call. They're how Claude connects to external services - databases, APIs, browsers, documentation sources.

The config lives in `.mcp.json` at your project root:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "some-mcp-package"],
      "env": {
        "API_KEY": "your-key"
      }
    }
  }
}
```

Each server entry tells Claude how to start the process (`command` + `args`) and what environment variables to pass (`env`).

**Important**: MCP servers only load when Claude starts. If you add or change `.mcp.json`, you must restart Claude (`exit` then `claude`).

## Exercise

### Step 1: Create .mcp.json

At your project root, create `.mcp.json`:

```json
{
  "mcpServers": {
    "Context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

Context7 provides up-to-date documentation for popular libraries. No API key needed.

### Step 2: Add WebSearch permissions

In `.claude/settings.local.json`, add to your `allow` list:

```json
"WebSearch",
"WebFetch(domain:github.com)",
"WebFetch(domain:docs.python.org)",
"WebFetch(domain:stackoverflow.com)"
```

WebSearch is a built-in Claude Code feature (not an MCP server). It lets Claude search the web and fetch pages from approved domains.

### Step 3: Keep secrets out of git

```bash
echo ".mcp.json" >> .gitignore
```

If your team needs a shared config, commit a template:

Create `.mcp.json.template`:
```json
{
  "mcpServers": {
    "Context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    },
    "your-api": {
      "command": "node",
      "args": ["path/to/server.js"],
      "env": {
        "API_KEY": "{{YOUR_API_KEY}}"
      }
    }
  }
}
```

### Step 4: Restart and test

```bash
# Exit current Claude session
exit

# Start fresh - MCP servers load on startup
claude
```

Ask Claude: "Using Context7, look up the latest FastAPI dependency injection docs"

## Verify It Works

1. Restart Claude after adding `.mcp.json`
2. Ask a question about a current library version: "What's new in Pydantic v2?"
3. Claude should use Context7 to fetch current docs rather than relying on training data
4. Ask Claude to search the web: "Search for the latest Python 3.13 release notes"
5. Claude should use WebSearch and return current information

## Other Useful MCP Servers

| Server | What It Does | Needs API Key? |
|--------|-------------|----------------|
| Context7 | Up-to-date library documentation | No |
| Playwright | Browser automation and testing | No |
| Sequential Thinking | Extended reasoning for complex problems | No |

**Sequential Thinking** config (no API key):
```json
{
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  }
}
```

**Playwright** config (no API key):
```json
{
  "playwright": {
    "command": "npx",
    "args": [
      "@playwright/mcp@latest",
      "--headless", "--browser", "chrome",
      "--viewport-size", "1920x1080"
    ]
  }
}
```

## What You Learned

- MCP servers live in `.mcp.json` at project root
- Each server is a process Claude launches and communicates with
- Servers load **only on startup** - restart Claude after changes
- Add `.mcp.json` to `.gitignore` (it may contain API keys)
- Commit a `.mcp.json.template` for team sharing
- Start with free, no-key servers (Context7, Sequential Thinking, Playwright)
- WebSearch is built-in (not an MCP server) and controlled via permissions

---

Next: [Automation](guide/05-automation.md)
