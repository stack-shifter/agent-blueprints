# Next.js — Agent Baseline

> STUB. The typing, style, testing, version-control, and boundary sections have not been
> filled in yet. Paste them from `snippets/`, or lift them from
> `stacks/typescript-node/agent.md`, which already carries them. Delete this note when done.

## Project shape

A Next.js application using the App Router. Routes live under `app/`, shared code under
`lib/` or `src/`, and components under `components/`. Read the tree before assuming more.

## Toolchain

- Dev server: `pnpm dev`
- Build: `pnpm build` — this is the real typecheck; it must pass clean
- Test: `pnpm test`
- Lint: `pnpm lint`

## Server and client boundaries

- Components are Server Components by default. Add `"use client"` only when the component
  needs state, effects, or browser APIs — and add it as low in the tree as possible.
- Never import server-only code (database clients, secrets, filesystem access) into a
  module that a client component can reach.
- Fetch data in Server Components. Do not fetch in a client component when the data can
  be resolved on the server.
- Mutations go through Server Actions or route handlers, and validate their input at the
  boundary. Treat every argument as untrusted.

## Strict typing

TODO — paste `snippets/typing/strict-typescript.md`.

## Code style

TODO — paste `snippets/style/vertical-code-layout.md`, `naming-conventions.md`, and
`comment-density.md`.

## Testing

TODO — paste `snippets/testing/test-first.md` and `no-mock-overuse.md`.

## Version control

TODO — paste `snippets/git/commit-message-format.md` and `branch-discipline.md`.

## Boundaries

TODO — paste `snippets/safety/ask-before-destructive.md` and `no-secret-exfiltration.md`.
