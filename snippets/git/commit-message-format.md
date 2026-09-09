## Commit messages

Format: `type(scope): summary` — `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`.

- Summary in the imperative mood, lower case, no trailing period, under ~70 characters. "add retry to fetch client", not "Added retries."
- The summary says what changed. The body, when present, says why — the problem, the constraint, the alternative rejected. Never restate the diff in prose.
- One logical change per commit. If the summary needs "and", split the commit.
- Never mix a refactor with a behavior change in one commit. They need different review attention.
- Reference the issue in the body, not the summary.
