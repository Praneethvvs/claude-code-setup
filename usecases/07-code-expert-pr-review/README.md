# 07: Code Expert / PR Review

## What It Does

A skill that turns Claude into a language-specific code expert with three modes:

| Mode | Trigger | What It Does |
|------|---------|-------------|
| **WRITE** | "create a service for...", "implement..." | Generates code following your project patterns |
| **REVIEW** | "review this file", "check this PR" | Checks against your coding standards |
| **DIAGNOSE** | "why is this failing?", "debug this error" | Traces root cause and suggests fix |

## Why It Matters

- Consistent code reviews against your team's actual standards (not generic best practices)
- New developers get instant feedback matching your patterns
- Build errors get diagnosed with project context

## Prerequisites

None. Works with any Python project.

## Setup Steps

### 1. Create the skill

```bash
mkdir -p .claude/skills/python
```

Create `.claude/skills/python/SKILL.md`:

```markdown
---
name: python
description: Python expert for {{ORG}} - Write, review, and diagnose code
model: claude-sonnet-4-5-20250929
---

# /python - Python Code Expert

## Modes

| Mode | Triggers |
|------|----------|
| WRITE | "write", "create", "implement", "add", "generate" |
| REVIEW | "review", "check", "look at", "PR review" |
| DIAGNOSE | "error", "failing", "broken", "debug", "fix" |

---

## CRITICAL RULES

### ALWAYS
1. Use type hints on all function signatures
2. Use Pydantic models for data validation
3. Use async/await for I/O operations
4. Use dependency injection (FastAPI Depends)
5. Write docstrings for public functions
6. Use `logging` module, never `print()`
7. Handle errors with custom exception classes
8. Write tests for new code

### NEVER
1. Never log PII or sensitive customer data
2. Never use `import *`
3. Never hardcode secrets or credentials
4. Never use bare `except:` (catch specific exceptions)
5. Never use mutable default arguments
6. Never skip input validation on API endpoints
7. Never commit `.env` files or credentials

---

## Code Patterns

### FastAPI Route
```python
from fastapi import APIRouter, Depends, HTTPException, status
from app.models.user import UserCreate, UserResponse
from app.services.user_service import UserService

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(
    user_data: UserCreate,
    service: UserService = Depends(),
) -> UserResponse:
    return await service.create(user_data)
```

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

### Repository Layer
```python
from sqlalchemy.ext.asyncio import AsyncSession

class UserRepository:
    def __init__(self, db: AsyncSession = Depends(get_db)):
        self.db = db

    async def get_by_email(self, email: str) -> User | None:
        result = await self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
```

### Custom Exceptions
```python
class AppError(Exception):
    def __init__(self, message: str, code: str):
        self.message = message
        self.code = code

class UserNotFoundError(AppError):
    def __init__(self, user_id: int):
        super().__init__(f"User {user_id} not found", "USER_NOT_FOUND")

class UserAlreadyExistsError(AppError):
    def __init__(self, email: str):
        super().__init__(f"User with this email exists", "USER_EXISTS")
```

### Test Pattern
```python
import pytest
from unittest.mock import AsyncMock

@pytest.fixture
def mock_repo():
    return AsyncMock(spec=UserRepository)

@pytest.fixture
def service(mock_repo):
    return UserService(repo=mock_repo)

class TestUserService:
    async def test_create_user_success(self, service, mock_repo):
        mock_repo.get_by_email.return_value = None
        mock_repo.create.return_value = User(id=1, email="test@{{org}}.com")

        result = await service.create(UserCreate(email="test@{{org}}.com"))

        assert result.id == 1
        mock_repo.create.assert_called_once()

    async def test_create_user_duplicate_raises(self, service, mock_repo):
        mock_repo.get_by_email.return_value = User(id=1, email="test@{{org}}.com")

        with pytest.raises(UserAlreadyExistsError):
            await service.create(UserCreate(email="test@{{org}}.com"))
```

---

## Code Review Checklist

- [ ] Type hints on all function signatures
- [ ] Pydantic models for request/response
- [ ] Async for all I/O operations
- [ ] Custom exceptions (not generic HTTPException everywhere)
- [ ] No PII in logs
- [ ] Input validation on API endpoints
- [ ] Tests for happy path and error cases
- [ ] No hardcoded secrets
- [ ] Dependency injection (not direct imports of singletons)
- [ ] Docstrings on public functions

## TRIGGER PHRASES
- `/python`
- `/python review`
- `/python write`
- `/python diagnose`
```

### 2. Use it

```
> /python review src/api/routes/users.py
> /python write a service for managing API keys
> /python diagnose: pytest test_users.py is failing with "async fixture not found"
```

## See Also

- [explanation.md](explanation.md) - How a code expert skill works, adapting for other languages
