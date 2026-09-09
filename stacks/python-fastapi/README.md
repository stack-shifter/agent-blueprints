# python-fastapi

Vault-internal notes. Not copied into projects — only `agent.md` is.

## Pick this when

The project is a FastAPI HTTP service.

## Assumes

- uv for environment and dependency management
- mypy strict, ruff for lint and format
- Pydantic models at every boundary
- A service layer between routes and persistence

## Pick a sibling instead when

There is no sibling Python baseline yet. For a Python CLI or library, copy this file,
strip the FastAPI section, and save it as a new stack directory rather than adding a
variant here.

## Status

Stub. Toolchain, type hints, and FastAPI conventions are written; style, testing, git, and
boundaries are TODOs pointing at snippets.
