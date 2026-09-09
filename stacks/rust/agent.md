# Rust — Agent Baseline

> STUB. The style, testing, version-control, and boundary sections have not been filled in
> yet. Paste them from `snippets/`. Delete this note when done.

## Project shape

A Rust binary or library. Source under `src/`, integration tests under `tests/`, unit
tests in a `#[cfg(test)]` module at the bottom of the file they cover.

## Toolchain

- Build: `cargo build`
- Test: `cargo test`
- Lint: `cargo clippy --all-targets -- -D warnings` — warnings are errors, fix them
- Format: `cargo fmt` — never hand-format against rustfmt
- Check fast: `cargo check`

## Types and errors

- No `unwrap()` or `expect()` outside tests and `main`. Propagate with `?` and let the
  caller decide. An `expect` that survives review needs a message stating the invariant
  that makes it unreachable.
- Libraries define their own error enum and implement `std::error::Error`. Binaries may
  use a boxed or anyhow-style error at the top level only.
- No `unsafe` without a `// SAFETY:` comment stating the invariant being upheld and why it
  holds here.
- Prefer borrowing to cloning. Reach for `clone()` when it is genuinely the simpler design,
  not to escape the borrow checker — if you are fighting it, the ownership model is wrong.
- Make illegal states unrepresentable with enums and newtypes rather than validating the
  same condition at every call site.
- Accept `&str` and `&[T]` in function signatures; return owned types.

## Async

- Do not introduce an async runtime into a crate that does not already have one.
- Never block inside an async context. Use the runtime's blocking pool for synchronous work.
- Hold no lock across an `.await`.

## Code style

TODO — paste `snippets/style/vertical-code-layout.md`, `naming-conventions.md`, and
`comment-density.md`.

## Testing

TODO — paste `snippets/testing/test-first.md` and `no-mock-overuse.md`.

## Version control

TODO — paste `snippets/git/commit-message-format.md` and `branch-discipline.md`.

## Boundaries

TODO — paste `snippets/safety/ask-before-destructive.md` and `no-secret-exfiltration.md`.
