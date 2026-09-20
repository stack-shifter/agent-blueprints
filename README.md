# agent-blueprints

A manual-copy vault for `AGENT.md` rules, sub-agent personas, and reusable rule blocks.

Not a sync tool, not a build system, no scripts. You open a file, copy it, and paste it
where you need it.

## The one rule

> **Every file here is complete and correct after a single copy.**

No copied artifact is symlinked or assembled at paste time. `CLAUDE.md` is the one
vault-internal exception: it points to `AGENTS.md` so repository guidance has one source.
No copied file inherits from a base layer, expects a fragment above or below it, or refers
to another path in this repo — a destination project cannot resolve one.

The cost is duplication: sibling baselines share most of their text. That is deliberate.
`snippets/` is what keeps the duplication consistent — it holds the canonical wording of
each rule, so the same rule reads identically everywhere it was pasted.

## Layout

Three directories, one per paste size.

| Directory | You copy | Into |
|---|---|---|
| `stacks/` | An entire `AGENT.md` for a project | `.claude/CLAUDE.md` |
| `sub-agents/` | One persona for one job | `.claude/agents/`, or a conversation |
| `snippets/` | One rule | An existing agent file |

Plus [`templates/`](templates/README.md), which holds the required shape for each of the
three, and explains how to instantiate one.

## Using it

**Starting a project.** Pick a stack, copy its `AGENT.md`:

```bash
mkdir -p .claude
cp ~/Code/stack-shifter/agent-blueprints/stacks/aws-cdk-serverless-api/AGENT.md .claude/CLAUDE.md
```

Then edit the copy freely. Project-specific rules belong in the copy, never upstream — the
vault holds only what is true for every project on that stack.

**Copies are snapshots.** Improving a baseline here does not reach projects already set up.
That is the trade for zero coupling. To propagate a change, re-copy deliberately and diff
against the destination's local edits.

**Reaching for a persona.** Pick the role, then take the file for your platform —
`sub-agents/claude/<role>.md` or `sub-agents/codex/<role>.toml`. Both exist for every
role, and each drops straight into that platform's agents directory.

**Composing rules.** `snippets/` works in two directions. Inward: when writing a stack
baseline, paste the snippet's text into it. Outward: when a live project needs one rule,
paste that snippet into its agent file. Always paste the text — never a path.

## Stacks

| Stack | Pick it when | Status |
|---|---|---|
| [`aspnetcore-rest-api`](stacks/aspnetcore-rest-api/) | A containerized ASP.NET Core REST API in C# — attribute-routed controllers, EF Core repositories, Postgres | **Complete** |
| [`aws-cdk-library`](stacks/aws-cdk-library/) | Working **on** a shared AWS CDK construct library that other repos build stacks with | **Complete** |
| [`aws-cdk-serverless-api`](stacks/aws-cdk-serverless-api/) | Building a production serverless API — API Gateway and Lambda, over DynamoDB or SQL — **with** that library | **Complete** |
| [`expressjs-rest-api`](stacks/expressjs-rest-api/) | A containerized Express 5 REST API — versioned routers, thin controllers, Zod validation, Drizzle repositories | **Complete** |
| [`react-vite-spa-lite`](stacks/react-vite-spa-lite/) | A React SPA where auth, styling, forms and data fetching are the project's own call | **Complete** |
| [`react-vite-spa-full`](stacks/react-vite-spa-full/) | A React SPA on the house set — Cognito hosted UI, Bootstrap 5, React Hook Form + Zod | **Complete** |
| [`angular-spa`](stacks/angular-spa/) | An Angular SPA — standalone components, signals, reactive forms, lazy routes | **Complete** |
| [`prototype-docs`](stacks/prototype-docs/) | A product docs and prototyping repo — canonical brief, numbered specs, self-contained HTML prototypes | **Complete** |

Stacks are **peers, never layers** — related ones sit side by side, never one under the
other. `aws-cdk-library` is for authoring the constructs and `aws-cdk-serverless-api` for
consuming them. `react-vite-spa-full` is a standalone superset of `react-vite-spa-lite`,
not an overlay to paste on top of it. `angular-spa` is deliberately a single baseline
rather than a pair: Angular ships forms, DI, routing and HTTP in the framework, so the
variable surface is too thin to justify splitting. If two baselines are so close that
choosing between them is unclear, they should be one baseline.

Each stack directory carries a `README.md` with what it assumes and when to prefer a
sibling. Those are vault-internal and never copied out.

## Sub-agents

Split by **platform**, because that is what decides the file format. Every role exists
twice — markdown for Claude Code, TOML for Codex — and **the two must say the same thing.**

| Role | Use it when | Claude | Codex |
|---|---|---|---|
| `authorization-boundary-auditor` | Auth, tenant ownership, object access, signed resources, or IAM grants changed | [md](sub-agents/claude/authorization-boundary-auditor.md) | [toml](sub-agents/codex/authorization-boundary-auditor.toml) |
| `contract-drift-auditor` | An API operation changed across providers, consumers, schemas, fixtures, or docs | [md](sub-agents/claude/contract-drift-auditor.md) | [toml](sub-agents/codex/contract-drift-auditor.toml) |
| `frontend-drift-auditor` | A change touched a shared shell, layout or token, and several screens must still agree | [md](sub-agents/claude/frontend-drift-auditor.md) | [toml](sub-agents/codex/frontend-drift-auditor.toml) |
| `infrastructure-change-reviewer` | CDK, CloudFormation, container, networking, IAM, or deployment behavior changed | [md](sub-agents/claude/infrastructure-change-reviewer.md) | [toml](sub-agents/codex/infrastructure-change-reviewer.toml) |
| `migration-safety-reviewer` | A schema, ORM, backfill, datastore, or API-version migration is ready for review | [md](sub-agents/claude/migration-safety-reviewer.md) | [toml](sub-agents/codex/migration-safety-reviewer.toml) |
| `spec-implementation-reconciler` | One approved product decision spans plans, prototypes, contracts, database docs, and code status | [md](sub-agents/claude/spec-implementation-reconciler.md) | [toml](sub-agents/codex/spec-implementation-reconciler.toml) |
| `spec-plan-reviewer` | An implementation must be validated against its delivery spec and phased plan before completion | [md](sub-agents/claude/spec-plan-reviewer.md) | [toml](sub-agents/codex/spec-plan-reviewer.toml) |
| `validation-review` | A phase claims to be done and you want it verified from a cold start | [md](sub-agents/claude/validation-review.md) | [toml](sub-agents/codex/validation-review.toml) |

`sub-agents/claude/*.md` copies into a project's `.claude/agents/`.
`sub-agents/codex/*.toml` copies into its `.codex/agents/`. Each works as-is where its
directory says it goes — no stripping, no conversion.

Review personas omit `Write`/`Edit` from their Claude `tools` and normally use
`sandbox_mode = "read-only"` in their Codex twins. `spec-plan-reviewer` is the narrow
exception: its Codex twin uses `workspace-write` so approved validation tools can create
temporary output, but its rules prohibit tracked changes and require cleanup. A persona
that must mutate tracked files carries `Write`/`Edit` and pairs it with `workspace-write`.

Shapes live in [`templates/sub-agent.template.md`](templates/sub-agent.template.md) and
[`templates/codex-agent.template.toml`](templates/codex-agent.template.toml).

**There is no `shared/` directory.** There was one, holding personas said to work on both
platforms — but Codex loads TOML, so a markdown file could never be dropped into
`.codex/agents/` and work. The name promised two platforms and delivered one. Two
directories named for the platform they actually load on keep the vault's single rule
intact: copy a file where its directory says, and it works.

## Snippets

Atomic rule blocks. One idea per file, no preamble, starting at `##` so a paste lands at
the right heading depth.

| Snippet | Rule |
|---|---|
| [`accessibility/wcag-aa-baseline`](snippets/accessibility/wcag-aa-baseline.md) | WCAG 2.2 AA: semantic elements, keyboard operability, accessible names, dialog semantics, live regions |
| [`style/prefer-boring-code`](snippets/style/prefer-boring-code.md) | Choose the implementation a mid-level developer reads once; abstract on the third occurrence, not the second |
| [`style/vertical-code-layout`](snippets/style/vertical-code-layout.md) | Reads top to bottom: one statement per line, guard clauses first, declare near first use |
| [`style/naming-conventions`](snippets/style/naming-conventions.md) | Names state what, not how; booleans as assertions; files named for their role in the layer |
| [`style/comment-density`](snippets/style/comment-density.md) | Comment why not what; match the peer group, and cover it uniformly |
| [`typing/strict-typescript`](snippets/typing/strict-typescript.md) | No `any`, no `as` to silence, no `!`; named domain types over catch-all maps |
| [`typing/python-type-hints`](snippets/typing/python-type-hints.md) | Every signature annotated; no bare `Any`; `X \| None` over `Optional` |
| [`testing/test-discipline`](snippets/testing/test-discipline.md) | Every behavior change ships with a test; one behavior each; never weaken an assertion for green |
| [`testing/verify-before-done`](snippets/testing/verify-before-done.md) | Run typecheck, lint, test, build before reporting done — scaled to the work |
| [`testing/no-mock-overuse`](snippets/testing/no-mock-overuse.md) | Mock external boundaries and deliberate seams; never mock implementation details |
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
the baselines where it applies. A persona proved out on one platform gets its twin written
for the other, so both stay paste-ready.

**Before committing**, confirm the one rule still holds: copy any single file into an empty
directory and read it cold. If it references a sibling path or assumes something was pasted
alongside it, fix the file.
