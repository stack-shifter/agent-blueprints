# typescript-node

Vault-internal notes. Not copied into projects — only `agent.md` is.

## Pick this when

The project is a TypeScript service, CLI, or library running on Node, with no framework
imposing its own conventions.

## Assumes

- `strict: true` in `tsconfig.json`
- Tests colocated with source as `*.test.ts`
- Scripts named `typecheck`, `test`, `lint`, `build` in `package.json`
- A schema validator available at the input boundaries

## Pick a sibling instead when

- **`typescript-nextjs`** — a Next.js app. Server/client boundaries and rendering rules
  matter enough that they change the advice, not just extend it.
- **`monorepo`** — the repo holds multiple packages. Reach for that baseline first and
  paste the typing and style sections from here into it.

## Status

Complete. This is the reference baseline — it is the worked example of the whole vault
pattern, assembled entirely by pasting from `snippets/`. When adding a new stack, read
this one first.
