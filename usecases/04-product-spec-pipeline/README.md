# 04: Product Spec to JIRA Pipeline

## What It Does

A chain of Claude commands that turns a product idea into structured JIRA tickets:

```
Idea --> /spec --> /requirements --> /to-jira --> /to-confluence
```

- `/spec` - Generates a structured product spec from an idea via interview
- `/requirements` - Captures exhaustive requirements (functional, data, UX, security)
- `/to-jira` - Converts the spec into JIRA Epic + Stories with acceptance criteria
- `/to-confluence` - Publishes the spec to Confluence

## Why It Matters

Product ideas get lost between Slack, meetings, and vague tickets. This pipeline forces structure and creates traceable artifacts: idea -> spec -> requirements -> tickets.

## Prerequisites

- JIRA/Confluence MCP server set up (see [03-jira-confluence-automation](../03-jira-confluence-automation/))

## Setup Steps

### 1. Create command files

```bash
mkdir -p .claude/commands
```

### 2. Create `/spec` command

Create `.claude/commands/spec.md`:

```markdown
---
description: "Create a product spec from a feature idea via interview"
---

## User Input
$ARGUMENTS

## Workflow

### Step 1: Understand
If input is vague, ask 3-5 targeted questions:
1. What problem are we solving?
2. Who is the user?
3. What does success look like?
4. Any hard constraints?

### Step 2: Generate Spec

# Product Specification: [FEATURE NAME]

**Status**: Draft
**Created**: [today]

## Problem Statement
[1-2 sentences: customer pain point]

## Target Users
| Role | Description | Impact |
|------|-------------|--------|
| [role] | [who] | [how feature affects them] |

## User Stories

### Story 1: [Title] (Must-Have)
**As a** [role], **I want** [capability], **So that** [benefit]
**Acceptance Criteria**:
- [ ] [testable criterion]

## Requirements
### Must Have
| ID | Requirement | Rationale |
|----|-------------|-----------|
| R-001 | [what] | [why] |

### Out of Scope
- [explicitly excluded items]

## Success Metrics
| Metric | Current | Target |
|--------|---------|--------|
| [metric] | [baseline] | [goal] |

## Risks
| Risk | Likelihood | Mitigation |
|------|------------|------------|
| [risk] | High/Med/Low | [how] |

### Step 3: Save
Save to: `specs/[feature-name]/product-spec.md`
```

### 3. Create `/to-jira` command

Create `.claude/commands/to-jira.md`:

```markdown
---
description: "Create JIRA Epic + Stories from a product spec"
---

## User Input
$ARGUMENTS

## Workflow

### Step 1: Read the spec
Find and read the spec file (from $ARGUMENTS or ask user).

### Step 2: Parse into tickets
- Feature name -> Epic summary
- Problem statement -> Epic description
- User stories -> Story tickets
- Requirements -> Story details
- Success metrics -> Acceptance criteria

### Step 3: Show for approval
ALWAYS show ticket details before creating:

## Tickets to Create
### Epic: [name]
### Stories:
| # | Summary | Priority | Requirements |
|---|---------|----------|-------------|

**Proceed?** (yes/no)

### Step 4: Create tickets
Use jira_create_issue. Create Epic first, then Stories linked to Epic.

### Step 5: Report
Show created ticket links and next steps.
```

### 4. Use it

```
> /spec User self-service password reset
[Claude interviews you, generates spec]

> /to-jira specs/password-reset/product-spec.md
[Claude creates Epic + Stories in JIRA]
```

## See Also

- [explanation.md](explanation.md) - Full pipeline details, templates, advanced options
