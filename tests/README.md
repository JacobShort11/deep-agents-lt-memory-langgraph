# Test Suite

Tests for the Deep Research Agent using mocked external services (no real API calls).

## Structure

```
tests/
├── conftest.py
├── unit/
│   ├── test_tools.py
│   ├── test_middleware.py
│   └── test_memory_management.py
└── integration/
    ├── test_subagents.py
    └── test_agent_orchestration.py
```

## Running Tests

```bash
pytest              # Run all tests
pytest -v           # Verbose output
pytest -m unit      # Unit tests only
pytest -m integration  # Integration tests only
```

## Markers

- `unit` - Fast, isolated tests
- `integration` - Tests for component interactions
- `requires_api` - Needs external API keys
