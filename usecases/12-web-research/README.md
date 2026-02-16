# 12: Web Research Integration

## What It Does

Claude can search the web and get up-to-date answers without leaving your terminal. Useful for:
- "What's the latest FastAPI best practice for dependency injection?"
- "How do I configure Azure App Service for Python 3.12?"
- "What changed in SQLAlchemy 2.1?"

## Why It Matters

Claude's training data has a cutoff date. For current documentation, release notes, or evolving best practices, web research bridges the gap.

## Prerequisites

None for basic. Perplexity API key for advanced.

## Setup Steps

### Option 1: Built-in WebSearch (Free, No Setup)

Just allow it in permissions:

```json
{
  "permissions": {
    "allow": [
      "WebSearch",
      "WebFetch(domain:github.com)",
      "WebFetch(domain:docs.python.org)",
      "WebFetch(domain:fastapi.tiangolo.com)",
      "WebFetch(domain:docs.microsoft.com)",
      "WebFetch(domain:docs.aws.amazon.com)",
      "WebFetch(domain:stackoverflow.com)"
    ]
  }
}
```

That's it. Claude can now search the web and fetch specific pages.

### Option 2: Perplexity MCP (Better Answers)

Perplexity provides AI-powered search results - more synthesized than raw web search.

1. Get API key: https://www.perplexity.ai/settings/api
2. Add to `.mcp.json`:

```json
{
  "mcpServers": {
    "perplexity-ask": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-e", "PERPLEXITY_API_KEY",
        "-e", "PERPLEXITY_ASK_MODEL",
        "mcp/perplexity-ask"
      ],
      "env": {
        "PERPLEXITY_API_KEY": "pplx-YOUR_KEY",
        "PERPLEXITY_ASK_MODEL": "sonar-pro"
      }
    }
  }
}
```

3. Allow in permissions:

```json
"mcp__perplexity-ask__perplexity_ask"
```

4. Restart Claude Code.

### Option 3: Context7 (Free, Library Docs)

Provides up-to-date documentation for popular libraries without an API key:

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

### Option 4: Deep Research (Comprehensive)

For in-depth research across multiple sources (costs ~$0.10 per query):

```json
{
  "mcpServers": {
    "deep-research": {
      "command": "node",
      "args": ["path/to/deep-research/dist/index.js"],
      "env": {
        "PERPLEXITY_API_KEY": "pplx-YOUR_KEY",
        "ANTHROPIC_API_KEY": "sk-ant-YOUR_KEY",
        "THINKING_BUDGET": "50000",
        "MAX_TOKENS": "64000"
      }
    }
  }
}
```

## Which to Use?

| Need | Tool | Cost |
|------|------|------|
| Quick factual lookup | WebSearch (built-in) | Free |
| Synthesized answer with sources | Perplexity | ~$0.008/query |
| Library documentation | Context7 | Free |
| Comprehensive multi-source research | Deep Research | ~$0.10/query |

Add this table to your CLAUDE.md so Claude picks the right tool:

```markdown
## Research Tools
| Need | Use | Cost |
|------|-----|------|
| Quick facts | WebSearch | Free |
| AI-synthesized answer | perplexity_ask | ~$0.008 |
| Library docs | Context7 | Free |
| Deep analysis | deep_research | ~$0.10 |
```

## See Also

- [explanation.md](explanation.md) - Comparison of research tools, when to use which
