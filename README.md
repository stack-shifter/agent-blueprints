# agent-blueprints

A manual-copy vault for `agent.md` rules, sub-agent personas, and reusable rule blocks.

Not a sync tool, not a build system, no scripts. You open a file, copy it, and paste it
where you need it.

## The one rule

> **Every file here is complete and correct after a single copy.**

Nothing is symlinked. Nothing is assembled at paste time. No file inherits from a base
layer or expects a fragment to be pasted above or below it, and no file that gets copied
out refers to another path in this repo — a destination project cannot resolve one.

The cost is duplication: sibling baselines share most of their text. That is deliberate.
`snippets/` is what keeps the duplication consistent — it holds the canonical wording of
each rule, so the same rule reads identically everywhere it was pasted.

## Layout

Three directories, one per paste size.

| Directory | You copy | Into |
|---|---|---|
| `stacks/` | An entire `agent.md` for a project | `.claude/CLAUDE.md` |
| `sub-agents/` | One persona for one job | `.claude/agents/`, or a conversation |
| `snippets/` | One rule | An existing agent file |

Plus `templates/`, which holds the required shape for each of the three.

## Using it

**Starting a project.** Pick a stack, copy its `agent.md`:

```bash
mkdir -p .claude
cp ~/Code/stack-shifter/agent-blueprints/stacks/rust/agent.md .claude/CLAUDE.md
```

Then edit the copy freely. Project-specific rules belong in the copy, never upstream — the
vault holds only what is true for every project on that stack.

**Copies are snapshots.** Improving a baseline here does not reach projects already set up.
That is the trade for zero coupling. To propagate a change, re-copy deliberately and diff
against the destination's local edits.

**Reaching for a persona.** Look in `sub-agents/shared/` first — that is where most roles
live. Check `claude/` or `codex/` only when platform capability actually matters.

**Composing rules.** `snippets/` works in two directions. Inward: when writing a stack
baseline, paste the snippet's text into it. Outward: when a live project needs one rule,
paste that snippet into its agent file. Always paste the text — never a path.

## Stacks

| Stack | Pick it when | Status |
|---|---|---|
| [`typescript-node`](stacks/typescript-node/) | A TypeScript service, CLI, or library on Node, no framework | **Complete** — the reference baseline |
| [`typescript-nextjs`](stacks/typescript-nextjs/) | A Next.js app on the App Router | Stub — server/client rules written |
| [`python-fastapi`](stacks/python-fastapi/) | A FastAPI HTTP service | Stub — toolchain and typing written |
| [`rust`](stacks/rust/) | A Rust binary or library | Stub — errors and async written |
| [`go`](stacks/go/) | A Go service or CLI | Stub — errors and design written |
| [`monorepo`](stacks/monorepo/) | Multiple packages in one workspace, any language | Stub — cross-cutting; combine with a language baseline |

Stubs carry their stack-specific rules and a `TODO — paste …` line for each remaining
section. Fill them by pasting from `snippets/`, then delete the stub banner.

Stacks are **peers, never layers**. `typescript-nextjs` sits beside `typescript-node`, not
under it. If two baselines are so close that choosing between them is unclear, they should
be one baseline.

Each stack directory may carry a `README.md` with what it assumes and when to prefer a
sibling. Those are vault-internal and never copied out.

## Sub-agents

Split by **platform**, not topic — so "can I paste this into Codex?" is answerable from the
path alone.

### `shared/` — works anywhere

| Persona | Use it when |
|---|---|
| [`security-auditor`](sub-agents/shared/security-auditor.md) | Changes touch auth, input parsing, I/O, or build a query or command |
| [`code-reviewer`](sub-agents/shared/code-reviewer.md) | Before opening a PR, or after a substantial change |
| [`test-builder`](sub-agents/shared/test-builder.md) | Coverage is missing, or before fixing a defect |
| [`debugger`](sub-agents/shared/debugger.md) | Something is broken and the cause is unknown |
| [`refactorer`](sub-agents/shared/refactorer.md) | Restructuring covered code without changing behavior |
| [`documentation-writer`](sub-agents/shared/documentation-writer.md) | Docs are missing or have drifted from the code |

### `claude/` — Claude Code only

| Persona | Use it when |
|---|---|
| [`repo-cartographer`](sub-agents/claude/repo-cartographer.md) | First contact with an unfamiliar codebase |
| [`migration-swarm`](sub-agents/claude/migration-swarm.md) | One proven mechanical change across many files |

### `codex/` — Codex / CLI only

| Persona | Use it when |
|---|---|
| [`patch-author`](sub-agents/codex/patch-author.md) | The model proposes a diff and a human applies it |

**The override rule.** A role name appears in `claude/` or `codex/` only if it is absent
from `shared/`, or is a deliberate override — and an override must open with a line saying
what it changes and why `shared/` was insufficient. Without this the two copies diverge
silently and you stop trusting either.

## Snippets

Atomic rule blocks. One idea per file, no preamble, starting at `##` so a paste lands at
the right heading depth.

| Snippet | Rule |
|---|---|
| [`style/prefer-boring-code`](snippets/style/prefer-boring-code.md) | Choose the implementation a mid-level developer reads once; abstract on the third occurrence, not the second |
| [`style/vertical-code-layout`](snippets/style/vertical-code-layout.md) | Reads top to bottom: one statement per line, guard clauses first, declare near first use |
| [`style/naming-conventions`](snippets/style/naming-conventions.md) | Names state what, not how; booleans as assertions; files named for their role in the layer |
| [`style/comment-density`](snippets/style/comment-density.md) | Comment why not what; match the peer group, and cover it uniformly |
| [`typing/strict-typescript`](snippets/typing/strict-typescript.md) | No `any`, no `as` to silence, no `!`; named domain types over catch-all maps |
| [`typing/python-type-hints`](snippets/typing/python-type-hints.md) | Every signature annotated; no bare `Any`; `X \| None` over `Optional` |
| [`testing/test-discipline`](snippets/testing/test-discipline.md) | Every behavior change ships with a test; one behavior each; never weaken an assertion for green |
| [`testing/verify-before-done`](snippets/testing/verify-before-done.md) | Run typecheck, lint, test, build before reporting done — scaled to the work |
| [`testing/no-mock-overuse`](snippets/testing/no-mock-overuse.md) | Mock only what you do not own; never mock internals |
| [`git/commit-message-format`](snippets/git/commit-message-format.md) | `type(scope): summary`, imperative, one logical change |
| [`git/branch-discipline`](snippets/git/branch-discipline.md) | Branch first; commit only when asked; never discard work you did not create |
| [`safety/ask-before-destructive`](snippets/safety/ask-before-destructive.md) | The explicit list of actions requiring a yes first |
| [`safety/no-secret-exfiltration`](snippets/safety/no-secret-exfiltration.md) | Never print, paste, or commit a secret value |

Improving a snippet propagates nowhere on its own. When you sharpen one, re-paste it into
the baselines already carrying the older phrasing — `grep -rl` a distinctive line from the
snippet across `stacks/` to find them.

## Adding to the vault

1. Start from the matching file in `templates/`.
2. Fill it in, delete the authoring-notes comment block.
3. Add a row to the table above. This index is the only file that must be updated when
   anything is added.

**Promotion path.** A rule proved out in a project becomes a snippet, then gets pasted into
the baselines where it applies. A persona drafted in `claude/` moves to `shared/` once you
confirm it needs no platform syntax.

**Before committing**, confirm the one rule still holds: copy any single file into an empty
directory and read it cold. If it references a sibling path or assumes something was pasted
alongside it, fix the file.
