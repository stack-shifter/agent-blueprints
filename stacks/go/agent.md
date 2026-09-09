# Go — Agent Baseline

> STUB. The style, testing, version-control, and boundary sections have not been filled in
> yet. Paste them from `snippets/`. Delete this note when done.

## Project shape

A Go service or CLI. Entry points under `cmd/`, internal packages under `internal/`.
Tests sit beside the code they cover as `*_test.go`.

## Toolchain

- Build: `go build ./...`
- Test: `go test ./...`
- Vet: `go vet ./...`
- Format: `gofmt -w` or `goimports` — never hand-format
- Lint: `golangci-lint run`

## Errors and design

- Handle every error. Never assign to `_` to silence one without a comment saying why it
  is safe.
- Wrap with context when propagating: `fmt.Errorf("read config: %w", err)`. The message
  says what was being attempted, not what failed.
- Compare errors with `errors.Is` and `errors.As`, never by string matching.
- No panics in library code. A panic is for a programmer error that cannot be recovered
  from, not for an unexpected input.
- Accept interfaces, return structs. Define the interface in the consuming package, not
  the implementing one, and keep it to the methods actually used.
- Every function taking a `context.Context` takes it as the first parameter and honors
  cancellation. Never store a context in a struct.
- Start no goroutine without a defined way for it to stop.

## Code style

TODO — paste `snippets/style/vertical-code-layout.md`, `naming-conventions.md`, and
`comment-density.md`. Note that Go's own conventions (short receiver names, no stuttering
in package-qualified identifiers) win where they conflict.

## Testing

TODO — paste `snippets/testing/test-first.md` and `no-mock-overuse.md`. Add a line
preferring table-driven tests with `t.Run` subtests.

## Version control

TODO — paste `snippets/git/commit-message-format.md` and `branch-discipline.md`.

## Boundaries

TODO — paste `snippets/safety/ask-before-destructive.md` and `no-secret-exfiltration.md`.
