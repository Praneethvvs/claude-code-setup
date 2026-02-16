# Explanation: Code Expert / PR Review

## How a Code Expert Skill Works

A production enterprise setup had an 860-line skill that turned Claude into a language-specific expert across 240+ projects. The key design elements:

### Three Modes with Auto-Detection

Claude detects the mode from natural language:
- "Write a service for..." -> WRITE mode (generate code)
- "Review this code..." -> REVIEW mode (check against standards)
- "Why is this failing..." -> DIAGNOSE mode (root cause analysis)

Each mode has a defined workflow:

**WRITE**: Understand requirements -> Find similar code in codebase -> Generate following patterns -> Build -> Test

**REVIEW**: Read code -> Check against 25+ checklist items -> Identify issues by severity (CRITICAL/HIGH/MEDIUM/LOW) -> Suggest fixes with code examples

**DIAGNOSE**: Reproduce the issue -> Parse error output -> Trace through code -> Propose fix -> Verify fix builds

### ALWAYS/NEVER Lists

The most impactful part. These rules catch real bugs. Examples for Python:

- **Always use type hints on function signatures** - catches type errors early
- **Never use bare `except:`** - swallows errors silently
- **Never use mutable default arguments** - shared state between calls
- **Never log PII or sensitive data** - compliance and security

### Full Code Patterns

The skill included complete, copy-paste-ready code examples for every layer:
- Service interface + implementation
- Repository interface + implementation
- Controller with proper attributes
- DI registration in correct order
- Exception handling with typed errors
- Unit tests with mocking

This meant Claude generated code that looked like it was written by a senior team member, because it followed the exact same patterns.

### Anti-Pattern Guide

Showed both the wrong way and the right way:

```python
# BAD: bare except swallows errors
try:
    result = do_something()
except:
    pass

# GOOD: catch specific, log, re-raise
try:
    result = do_something()
except ValueError as e:
    logger.error("Validation failed", exc_info=True)
    raise
```

## Claude Features That Enable This

### Skills

Skills are the right choice (not commands) because:
- Code expertise requires **deep context** (patterns, conventions, anti-patterns)
- The skill is invoked **frequently** across many different files
- It needs **multiple modes** (write/review/diagnose)
- It benefits from a **specific model** (Opus for thorough review)

### Model Selection

For code review and writing, `claude-sonnet-4-5-20250929` is the recommended model. It's faster and cheaper than Opus while being excellent at code tasks. Reserve Opus (`claude-opus-4-6`) only for complex architectural analysis where thoroughness matters more than speed.

### File Pattern Triggers

```yaml
triggers:
  file_patterns:
    - "**/*.py"
```

When Claude encounters Python files, this skill is automatically suggested.

## Adapting for {{ORG}}

### Customize the Patterns

Replace the example patterns in the README with your actual project patterns. Read through your best code and extract:

1. **How do you structure API routes?** (FastAPI routers, decorators, response models)
2. **How do you structure services?** (dependency injection, error handling)
3. **How do you structure tests?** (fixtures, mocking, naming)
4. **What are your naming conventions?** (files, classes, functions, variables)

### Add {{ORG}}-Specific Rules

```markdown
### ALWAYS
- Use Ruff for formatting (not Black)
- Use SQLAlchemy 2.0 style (not 1.x legacy)
- Use `datetime.UTC` not `datetime.utcnow()`
- Return Pydantic models from API endpoints, never dicts

### NEVER
- Never use `requests` library (use `httpx` for async)
- Never use `os.environ` directly (use Pydantic Settings)
- Never write raw SQL (use SQLAlchemy ORM)
```

### Multiple Language Skills

If {{ORG}} has multiple languages, create separate skills:

```
.claude/skills/
├── python/SKILL.md      # Python expert
├── terraform/SKILL.md   # Infrastructure expert
└── typescript/SKILL.md  # Frontend expert (if applicable)
```

Each skill has its own patterns, rules, and checklist tailored to that language's role in your stack.
