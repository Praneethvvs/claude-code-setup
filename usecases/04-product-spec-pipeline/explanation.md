# Explanation: Product Spec to JIRA Pipeline

## How the Original Enterprise Setup Worked

A production setup used **six command files** forming a full product management pipeline:

| Command | Purpose |
|---------|---------|
| `/product.spec` | Generate spec from idea (interview-driven) |
| `/product.requirements` | Deep-dive requirements across 6 categories |
| `/product.to-jira` | Convert spec to JIRA Epic + Stories |
| `/product.from-confluence` | Import existing Confluence page as spec |
| `/product.to-confluence` | Publish spec to Confluence |
| `/compose` | Intelligent multi-commit composition (separate use case) |

### The Requirements Command (Deep Dive)

The `/product.requirements` command was particularly thorough. It interviewed across **6 requirement categories**:

1. **Functional** - What must the system do? Actions, inputs, outputs, edge cases
2. **Data** - CRUD operations, field types, validation rules, migration needs
3. **User Experience** - Discovery, entry points, feedback, error states, accessibility
4. **Integration** - Affected systems, APIs consumed/exposed, third-party deps
5. **Performance** - Response time, load, concurrency, data volume limits
6. **Security & Compliance** - Permissions, sensitive data handling, audit logging, data retention

Each requirement got a unique ID (FR-001, DR-001, UX-001, etc.) and was tracked in a **traceability matrix**:

```
| Requirement | User Story | Test Case | JIRA Ticket |
|-------------|------------|-----------|-------------|
| FR-001      | US-001     | TC-001    | {{ORG}}-123    |
```

### The Spec-to-JIRA Conversion

The `/product.to-jira` command had smart project routing:

| Keyword in Spec | JIRA Project |
|----------------|-------------|
| Portal, Dashboard | {{ORG}}-WEB |
| API, Backend | {{ORG}}-API |
| Pipeline, Data | {{ORG}}-DATA |
| Unclear | Ask user |

And strict safety rules:
- **Always show for approval before creating** - Claude displays the full ticket list in chat
- **Never create Subtasks** - use Task with Parent-Child link
- **Include requirements IDs** - link each Story back to specific requirements (R-001, R-002)

### Confluence Round-Trip

The pipeline supported both directions:
- **Import**: Read existing Confluence page -> convert to spec format -> identify gaps
- **Export**: Take spec -> convert to Confluence markup -> create/update page -> add labels

Gap analysis was particularly useful: after importing a Confluence page, Claude would report:
```
Content Found:
- [x] Problem statement
- [x] User stories
- [ ] Acceptance criteria (MISSING)
- [ ] Success metrics (MISSING)
```

## Claude Features That Enable This

### Commands

Commands are simpler than skills - just markdown instructions with `$ARGUMENTS` placeholder. They're perfect for one-shot workflows like "convert this spec to tickets."

The dot notation (`product.spec`, `product.to-jira`) creates a command namespace:
```
.claude/commands/
├── product.spec.md          -> /product.spec
├── product.requirements.md  -> /product.requirements
├── product.to-jira.md       -> /product.to-jira
├── product.from-confluence.md -> /product.from-confluence
└── product.to-confluence.md  -> /product.to-confluence
```

### MCP Server (Atlassian)

The read and write JIRA/Confluence tools power the create/import/export operations.

### CLAUDE.md Guardrails

Critical rules prevent mistakes:
- Always show for approval before creating tickets
- Never include code snippets in JIRA descriptions (business-friendly language only)
- Always link to parent Epic

## Adapting for {{ORG}}

### Simplify to Start

You don't need all 6 commands. Start with two:

1. `/spec` - Interview + generate spec
2. `/to-jira` - Convert spec to tickets

Add the others as needs arise.

### Customize the Spec Template

Edit the template sections to match {{ORG}}'s needs. If you don't deal with compliance, drop that section. If you need infrastructure planning, add it.

### Add Project Routing

Update the project routing logic for your JIRA projects:

```markdown
| Content Area | JIRA Project |
|-------------|-------------|
| Python API services | {{ORG}}-API |
| Frontend/UI | {{ORG}}-WEB |
| Infrastructure | {{ORG}}-INFRA |
| Data/ML | {{ORG}}-DATA |
```
