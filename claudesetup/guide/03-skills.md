# Skills

> Time: ~15 min | You will learn: **how skills work** | You will build: **a Python code expert skill**

## What You're Building

A skill that turns Claude into a Python code expert with three modes: WRITE (generate code matching your patterns), REVIEW (check code against your standards), and DIAGNOSE (trace errors with project context).

## The Concept

Skills are specialist personas. Where commands are one-shot workflows ("do this sequence of steps"), skills give Claude deep domain knowledge, rules, and multiple modes of operation.

```
.claude/skills/
├── python/
│   └── SKILL.md        # Invoked as /python
└── security/
    └── SKILL.md        # Invoked as /security
```

A skill file has frontmatter and a structured body:

```markdown
---
name: skill-name
description: What this skill does (shown in skill list)
model: claude-sonnet-4-5-20250929
---

# /skill-name - Title

## Modes (optional - what the skill can do)
## CRITICAL RULES (ALWAYS / NEVER lists)
## WORKFLOW (steps the skill follows)
## OUTPUT FORMAT (structured output template)
## TRIGGER PHRASES (how to invoke it)
```

**Model selection** in the frontmatter controls which Claude model runs the skill:

| Model | Best For | Speed | Cost |
|-------|---------|-------|------|
| `claude-opus-4-6` | Complex analysis, planning, deep research | Slow | High |
| `claude-sonnet-4-5-20250929` | Code writing, review, implementation | Medium | Medium |
| `claude-haiku-4-5-20251001` | Simple queries, formatting, triage | Fast | Low |

## Exercise

### Step 1: Create the skill

```bash
mkdir -p .claude/skills/python
```

Create `.claude/skills/python/SKILL.md`:

```markdown
---
name: python
description: Python expert - write, review, and diagnose code
model: claude-sonnet-4-5-20250929
---

# /python - Python Code Expert

## Modes

| Mode | Triggers |
|------|----------|
| WRITE | "write", "create", "implement", "add" |
| REVIEW | "review", "check", "look at", "PR review" |
| DIAGNOSE | "error", "failing", "broken", "debug", "fix" |

---

## CRITICAL RULES

### ALWAYS
1. Use type hints on all function signatures
2. Use Pydantic models for data validation
3. Use async/await for I/O operations
4. Write docstrings for public functions
5. Use `logging` module, never `print()`
6. Handle errors with specific exception classes
7. Write tests for new code

### NEVER
1. Never use `import *`
2. Never hardcode secrets or credentials
3. Never use bare `except:` (catch specific exceptions)
4. Never use mutable default arguments
5. Never skip input validation on API endpoints
6. Never commit `.env` files

---

## Code Patterns

### Service Layer
```python
from app.repositories.user_repo import UserRepository

class UserService:
    def __init__(self, repo: UserRepository = Depends()):
        self.repo = repo

    async def create(self, data: UserCreate) -> User:
        existing = await self.repo.get_by_email(data.email)
        if existing:
            raise UserAlreadyExistsError(data.email)
        return await self.repo.create(data)
```

### Test Pattern
```python
import pytest
from unittest.mock import AsyncMock

@pytest.fixture
def mock_repo():
    return AsyncMock(spec=UserRepository)

class TestUserService:
    async def test_create_success(self, service, mock_repo):
        mock_repo.get_by_email.return_value = None
        result = await service.create(UserCreate(email="test@example.com"))
        assert result.id == 1
```

---

## Code Review Checklist

When in REVIEW mode, check:
- [ ] Type hints on all function signatures
- [ ] Pydantic models for request/response
- [ ] Async for all I/O operations
- [ ] Specific exceptions (not generic HTTPException everywhere)
- [ ] Input validation on API endpoints
- [ ] Tests for happy path and error cases
- [ ] No hardcoded secrets
- [ ] Docstrings on public functions

---

## TRIGGER PHRASES
- `/python`
- `/python review`
- `/python write`
- `/python diagnose`
```

### Step 2: Customize the rules

The ALWAYS/NEVER lists above are examples. **Replace them with your team's actual standards.** The code patterns section should use patterns from your real codebase.

### Step 3: Try it

```
> /python review src/api/routes/users.py
> /python write a service for managing API keys
> /python diagnose: pytest test_users.py is failing with "async fixture not found"
```

## Verify It Works

1. Run `/python review` on a file in your project
2. Claude should check against the ALWAYS/NEVER rules and the review checklist
3. Run `/python write a utility function for date parsing`
4. Claude should generate code following the patterns defined in the skill

## What You Learned

- Skills live in `.claude/skills/<name>/SKILL.md`
- Skills define a **persona** with rules, modes, and domain knowledge
- ALWAYS/NEVER lists are the most powerful part - they encode your team's standards
- The `model` frontmatter controls which Claude model runs the skill
- **Commands = workflows** (do steps A, B, C). **Skills = expertise** (know rules, patterns, multiple modes)

---

Next: [MCP Servers](guide/04-mcp-servers.md)
