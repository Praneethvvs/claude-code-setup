# Explanation: Web Research Integration

## How the Original Enterprise Setup Worked

A production setup can configure multiple tiers of research tools and tell Claude which to prefer:

### Tier 1: Perplexity Ask (~$0.008/query)

A Docker-based MCP server using the `sonar-pro` model. Fast (~5 seconds), good for factual lookups:
- "What's the latest Azure SDK version for Python?"
- "How does Auth0 handle token refresh in SPAs?"
- "What are the default timeout settings for Redis 7?"

### Tier 2: Deep Research (~$0.10/query)

A Node.js MCP server combining Perplexity search with Anthropic analysis. Slower (~30 seconds), comprehensive:
- "Compare approaches to implementing distributed transactions in Azure"
- "What are the security implications of each OAuth2 flow for our architecture?"
- "Analyze the trade-offs between ECS Fargate and AKS for our workload"

### Critical Rule in CLAUDE.md

```markdown
## MCP Tools - Research
- DO NOT use WebSearch - use perplexity_ask or deep_research
```

This told Claude to prefer the specialized tools over the built-in web search. The reason: Perplexity and Deep Research provide better-synthesized answers with source attribution.

### Sequential Thinking

For complex problems requiring multi-step reasoning, they also had:

```json
{
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  }
}
```

This gave Claude an extended thinking scratchpad for breaking down complex problems step-by-step.

### Context7

For up-to-date library documentation:

```json
{
  "Context7": {
    "command": "npx",
    "args": ["-y", "@upstash/context7-mcp@latest"]
  }
}
```

This pulls current documentation for popular frameworks, ensuring Claude doesn't use outdated API patterns.

## Research Tool Comparison

| Feature | WebSearch | Perplexity | Context7 | Deep Research |
|---------|-----------|------------|----------|--------------|
| Cost | Free | ~$0.008 | Free | ~$0.10 |
| Speed | ~3s | ~5s | ~3s | ~30s |
| Depth | Surface | Moderate | Library-specific | Comprehensive |
| Sources | Links | AI summary + links | Official docs | Multi-source analysis |
| Setup | None | API key + Docker | npx | API keys + Node.js |
| Best for | Quick facts | Technical questions | "How do I do X in Y?" | Architectural decisions |

## Adapting for {{ORG}}

### Minimum Setup (Free)

Just allow WebSearch and WebFetch in permissions. This gives Claude basic web access:

```json
"WebSearch",
"WebFetch(domain:github.com)",
"WebFetch(domain:docs.python.org)",
"WebFetch(domain:docs.aws.amazon.com)",
"WebFetch(domain:docs.microsoft.com)"
```

### Recommended Setup

Add Perplexity + Context7 for better answers:

1. Perplexity ($5/month for moderate use) - best overall quality
2. Context7 (free) - keeps library docs current
3. Sequential Thinking (free) - better reasoning for complex questions

### CLAUDE.md Research Rules

Add to your CLAUDE.md:

```markdown
## Research Tools

When you need to look something up:
1. For library/framework docs: use Context7
2. For technical questions: use perplexity_ask
3. For deep architectural analysis: use deep_research
4. For reading a specific URL: use WebFetch
5. Fallback: use WebSearch

Always cite sources when sharing research results.
```

### Domain Allow-List for WebFetch

Only allow domains you trust:

```json
"WebFetch(domain:github.com)",
"WebFetch(domain:docs.python.org)",
"WebFetch(domain:fastapi.tiangolo.com)",
"WebFetch(domain:docs.pydantic.dev)",
"WebFetch(domain:docs.sqlalchemy.org)",
"WebFetch(domain:docs.microsoft.com)",
"WebFetch(domain:docs.aws.amazon.com)",
"WebFetch(domain:stackoverflow.com)",
"WebFetch(domain:pypi.org)"
```

This prevents Claude from fetching arbitrary URLs while giving it access to the documentation sites you use most.
