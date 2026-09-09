# Python / FastAPI — Agent Baseline

> STUB. The style, testing, version-control, and boundary sections have not been filled in
> yet. Paste them from `snippets/`. Delete this note when done.

## Project shape

A FastAPI service. Application code lives under `app/` or `src/`, tests under `tests/`.
Routes are thin: they validate, delegate to a service layer, and serialize. Business logic
does not live in a route handler.

## Toolchain

- Environment and dependencies: `uv sync`
- Run: `uv run uvicorn app.main:app --reload`
- Test: `uv run pytest`
- Typecheck: `uv run mypy .` — must pass clean before a change is done
- Lint and format: `uv run ruff check .` and `uv run ruff format .`

Never edit a lockfile by hand. Never `pip install` into the environment directly.

## Type hints

Every function signature is annotated — parameters and return type, including `-> None`.

- The codebase typechecks clean under strict settings. Code that only passes with checks relaxed does not ship.
- Never use bare `Any`. When a type is genuinely open, use a protocol or a type variable. If `Any` is truly unavoidable, comment why.
- No `# type: ignore` without a trailing comment naming the reason and the upstream issue.
- Prefer `X | None` over `Optional[X]`, and built-in generics (`list[str]`, `dict[str, int]`) over the `typing` aliases.
- Use `Protocol` for structural interfaces rather than inheriting from an ABC purely to satisfy a type.
- Validate external input at the boundary with a model, and let the model's type flow inward.

## FastAPI conventions

- Every request and response body is a Pydantic model. No raw `dict` in or out of a route.
- Declare the response model on the decorator so the schema stays honest.
- Use dependency injection for database sessions, auth, and configuration. Do not reach for a global.
- `async def` routes must not call blocking code. If a library is synchronous, run it in a thread pool.
- Raise `HTTPException` at the boundary only. Internal layers raise domain exceptions that the boundary translates.

## Code style

TODO — paste `snippets/style/vertical-code-layout.md`, `naming-conventions.md`, and
`comment-density.md`.

## Testing

TODO — paste `snippets/testing/test-first.md` and `no-mock-overuse.md`.

## Version control

TODO — paste `snippets/git/commit-message-format.md` and `branch-discipline.md`.

## Boundaries

TODO — paste `snippets/safety/ask-before-destructive.md` and `no-secret-exfiltration.md`.
Add a line about migrations: they are generated, reviewed, and never auto-applied.
