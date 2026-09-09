## Type hints (Python)

Every function signature is annotated — parameters and return type, including `-> None`.

- The codebase typechecks clean under strict settings. Code that only passes with checks relaxed does not ship.
- Never use bare `Any`. When a type is genuinely open, use a protocol or a type variable. If `Any` is truly unavoidable, comment why.
- No `# type: ignore` without a trailing comment naming the reason and the upstream issue.
- Prefer `X | None` over `Optional[X]`, and built-in generics (`list[str]`, `dict[str, int]`) over the `typing` aliases.
- Use `Protocol` for structural interfaces rather than inheriting from an ABC purely to satisfy a type.
- Validate external input at the boundary with a model, and let the model's type flow inward.
