# AWS CDK Construct Library — Agent Baseline

Rules for working in a published AWS CDK construct library. They apply to every file
unless a more specific instruction in the task overrides them.

This library is consumed by other teams' CDK stacks. Everything public is a contract, and
a careless rename replaces live infrastructure in someone else's account.

**Assumed majors:** Node 24, TypeScript 7, AWS CDK 2. Only a few rules below
depend on a major, and each says so where it does. If the project is on a different major,
the project wins — change this file rather than follow a rule that no longer applies.

## Project shape

Two audiences, one package. `src/iac/` holds constructs used inside a CDK stack;
`src/framework/` holds utilities imported inside the Lambda handler at runtime. Keep the
split — a module that reaches across it belongs in neither.

`src/main.ts` is the only entry point. The package `exports` map has a single `"."` key,
so deep imports are blocked by Node itself. Anything public must be re-exported from the
barrel, and anything re-exported from the barrel is public — widening it is an API change.

Follow the project's established test layout. Build output is generated; never edit it by
hand.

## Toolchain

Use npm. Read `package.json` before running anything: install flags, script names, output
paths, and composed checks are project configuration. Run every configured build, test,
typecheck, format-check, and package-validation step that applies. If one is missing,
report it rather than inventing an equivalent.

## Prefer boring code

When several implementations work, choose the one a mid-level developer understands on the first read. Cleverness is a cost paid by every future reader.

- Prefer plain control flow — `if`, `for`, early returns — over deeply chained pipelines or condensed one-liners. Favor code that can be stepped through in a debugger.
- Follow the patterns already in the repository. Do not introduce a second architectural style alongside a working one.
- Some duplication is acceptable when it keeps logic readable and local. Do not abstract on the second occurrence; wait for the third.
- Build only what the current task requires. Never generalize for a hypothetical future caller.
- Do not introduce factories, builders, dependency-injection layers, or similar patterns unless the problem clearly demands one.
- Do not add a dependency for something the standard library already does.
- Added complexity is fine when it buys correctness, performance, or maintainability — but comment why it was worth the cost.

## Strict typing

`strict: true` is assumed. Code that only compiles with it off does not ship.

- Never use `any`. When a type is genuinely unknown, use `unknown` and narrow it explicitly.
- Never use `as` to silence an error. A cast is a claim you cannot prove; fix the type instead. The only acceptable casts are on parsed external input immediately after validation.
- Never use non-null assertions (`!`). Narrow with a check, or make the type honest about nullability.
- No `@ts-ignore`. `@ts-expect-error` is permitted only with a comment naming the upstream issue it works around.
- Type the boundaries, infer the interior. Annotate exported signatures explicitly; let inference handle local variables.
- Prefer discriminated unions over optional-field soup. Make illegal states unrepresentable rather than checking for them.
- Prefer named domain types over catch-all maps. Reserve `Record<string, unknown>` for genuinely untyped external payloads — never for config objects, query-expression maps, or any structure whose keys you know or can partially enumerate.
- Validate all external input (network, filesystem, env, user) at the boundary with a schema, and type the interior from the schema's output.

Read the project's TypeScript configuration before relying on an optional compiler check.
Do not copy flag values or workarounds from another construct library; the local compiler
configuration and its diagnostics are authoritative.

## Construct API design

This section is the reason the library exists. Everything here is a contract.

- Extend `Construct`, never the CDK L2 itself. Expose the wrapped L2 as a
  `public readonly` field — that field *is* the escape hatch, and it is why consumers
  never need you to add a passthrough prop.
- Narrow the native props type rather than replacing it: extend the CDK props interface
  and promote the few options you require. Never retype fields the CDK already defines.
- Use native CDK property names and native CDK types throughout. Invent a name only where
  there is genuinely no native equivalent.
- Derive types with `Pick` and `Partial` from the native props instead of hand-copying
  field lists that will drift.
- Keep every IaC interface, enum, and deprecated alias in one models file. Do not scatter
  props back into the construct files.
- Name constructor props `<Thing>Props` and method arguments `<Thing>Options`.

**The merge contract.** When route options merge with defaults: scalars overwrite,
`environment` and `bundling` deep-merge, and `grants` concatenate with the defaults first.
Changing any of those three behaviors is a breaking change. Test all three.

- Check `inheritDefaults === false` strictly. `!inheritDefaults` also catches `undefined`
  and silently disables inheritance for callers who never opted out.
- A tri-state option distinguishes "inherit" from "explicitly none" by key presence, not
  truthiness. Use `hasOwnProperty`; `options.x ?? defaults.x` collapses the two and breaks
  the caller who passed `null` on purpose.
- Grant IAM only through grant callbacks. This library never constructs a
  `PolicyStatement` — the consumer owns their own permissions.
- Deprecate by adding an aliased type with a `@deprecated` tag. Never delete a public name.

**Two rules whose violation is invisible at compile time:**

- **Nested construct IDs are CloudFormation logical-ID inputs.** The literal string you
  pass as a child construct's id, and any value used as a construct id, appears in the
  synthesized logical ID. Renaming one replaces that resource in every consumer's deployed
  stack. Treat any such rename as breaking, and say so in the commit.
- **When you add a field to a route options interface, add it to the destructure
  strip-list in the same commit.** Options are destructured to separate library-specific
  fields from the native Lambda props before the rest is spread through. A field you
  forget to strip leaks into the native props, and the cast on that call means the
  compiler stays silent. This is the most likely regression in this repository.

## Vertical code layout

Write code that reads top to bottom like prose. Density is not concision.

- One statement per line. Never chain unrelated operations onto one line to save space.
- Separate logical stanzas with a single blank line. Group a variable's declaration with the lines that first use it.
- Declare variables close to first use, not in a block at the top of the function.
- Put the early returns and guard clauses first, then the main path. Never wrap the main path in an `else`.
- Order functions so a caller appears above the functions it calls, so the file reads at descending levels of detail.
- Break a long expression across lines at the operators, aligning the operands — do not let it wrap arbitrarily at the margin.
- Keep functions short enough to see whole. If you cannot see the top and bottom at once, extract a named helper.

## Naming

- Names state what a thing is or does, not how it is implemented. `activeUsers`, not `filteredArray`.
- No abbreviations except ones universal in the domain (`id`, `url`, `db`). Never invent one to save characters.
- Booleans read as assertions: `isReady`, `hasAccess`, `shouldRetry`. Never negated in the name — `isDisabled` over `isNotEnabled`.
- Functions that do something are verb phrases; functions that return something are noun phrases or `getX`. Do not mix both roles in one function.
- The length of a name scales with the size of its scope. A loop index may be `i`; a module-level export may not.
- Name files for their role in the layer they sit in: `client.handler.ts`, `clients.controller.ts`, `client.repository.ts`, `client.validation.ts`. A test file takes the name of the unit under test plus the suffix the project uses.
- Match the surrounding file's existing vocabulary. If the codebase says `account`, do not introduce `user` for the same concept.

## Comments

Comment why, never what. If a comment restates the code, delete the comment or fix the name.

- Write a comment when the reason for the code is not recoverable from the code: a workaround, a non-obvious constraint, an ordering that matters, a deliberate deviation.
- Never leave commented-out code. Version control already remembers it.
- No changelog comments, no attribution comments, no `// TODO` without a concrete next action.
- Do not decide comment style per file. Match the closest existing peer in the same layer — a heavily annotated codebase and a bare one both have a house style.
- Keep coverage uniform within a peer group. Documenting some exported functions in a module while leaving equivalent ones bare is worse than documenting none.
- Doc comments on exported symbols state contract, not implementation: what it takes, what it returns, what it throws, what it assumes.

Doc comments are unusually load-bearing here, because they are what a consumer reads in
their editor instead of the source:

- Document every exported symbol, every interface member, and every enum member. Document
  private members too.
- One imperative sentence, ending in a period.
- `@default` takes a bare value with no backticks or quotes.
- `@param name - Description.` with the spaced hyphen, then `@returns Description.`

## Testing

Every behavior change ships with a test that covers it. Tests follow the project's established layout and are named for the unit under test.

- One behavior per test. The test name states the behavior in a sentence: `returns empty list when no matches`, not `search 2`.
- Write tests before or after the implementation, whichever suits the work. Ordering is not the point; covering the changed behavior before reporting done is.
- Confirm every new test can actually fail. A test written against already-passing code may be asserting nothing — revert the change, watch it go red, restore it.
- For a bug fix, that check is mandatory: a test that has never failed cannot prove it covers the reported bug.
- Assert on observable behavior, not on internal calls. A test that breaks on a refactor with no behavior change is a bad test.
- Cover the boundaries: empty, one, many, and the error path. The happy path alone is not coverage.
- Keep tests deterministic. Stub time, randomness, and external side effects rather than tolerating a flaky assertion.
- Never weaken an assertion or delete a test to make a suite green. If a test is wrong, say so explicitly and explain why before changing it.
- Never report work as done on an unrun suite. If tests cannot be run, say that plainly rather than implying they passed.

Mock at external boundaries and deliberate architectural seams, nowhere else.

- Mock what you do not own: network calls, clocks, randomness, the filesystem, and third-party services. Mock an internal collaborator only through an intentional seam such as an injected repository or composition root.
- Never mock the thing under test or an internal implementation detail merely to make a test easier to write. That couples the test to the implementation and it will pass while the code is broken.
- Prefer a real in-memory implementation (an actual sqlite, a fake repository) over a mock with scripted return values.
- If a test needs more than a couple of mocks to run, the design is telling you the unit has too many dependencies. Fix the design instead of adding mocks.
- Never assert that a mock was called as the primary assertion. Assert on the resulting state or output.

For construct tests specifically:

- Build a fresh stack inside each test, then assert against the synthesized template.
  Never share a stack between tests; CDK state leaks.
- Assert a default *did not* apply with an absence matcher. An opt-out with no absence
  assertion is untested.
- Assert grants by call count and by identity — that the same function instance reached
  each grant. Count alone passes when the wrong function was granted.
- Cover four axes for every construct: defaults, overrides, failure paths, permissions.

## Verification

Run the project's checks before reporting a change complete. Never describe work as done on checks you did not run.

- The default gauntlet is typecheck, lint, test, build. Run all of it after any change to runtime code.
- Scale the checks to the work. A documentation-only or comment-only change does not need the full suite — say which checks you skipped and why.
- Run the checks the repository actually defines. If a script is missing, say so rather than substituting an equivalent command.
- Fix what the checks report before moving on. Never leave a failing check for the reviewer to find.
- If a check cannot run in this environment, name it and say so plainly. Do not imply it passed.
- Report the outcome, not the intent: name the commands you ran and what they returned.

Read CI configuration for the required checks and their order; do not restate a snapshot of
that pipeline here.

## Version control

Format: `type(scope): summary` — `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`.

- Summary in the imperative mood, lower case, no trailing period, under ~70 characters. "add retry to fetch client", not "Added retries."
- The summary says what changed. The body, when present, says why — the problem, the constraint, the alternative rejected. Never restate the diff in prose.
- One logical change per commit. If the summary needs "and", split the commit.
- Never mix a refactor with a behavior change in one commit. They need different review attention.
- Reference the issue in the body, not the summary.

- Never commit directly to the default branch. Branch first, always.
- Branch names: `type/short-description` in kebab-case — `fix/token-refresh-race`.
- Commit and push only when explicitly asked. Finishing a change is not permission to commit it.
- Never amend, rebase, force-push, or reset a branch that has been pushed, unless explicitly told to.
- Never use `git checkout .`, `git reset --hard`, or `git clean` to discard work you did not create. Uncommitted changes in the tree may be the user's.
- Never commit generated artifacts, dependency directories, or anything matched by the ignore file. Never commit a credential, even to a private repo.

**Never hand-edit the package version.** It stays fixed in the repository and the release
workflow sets it from the release tag. A manual bump produces a version that disagrees
with the tag.

## Boundaries

Stop and ask before any action that is hard to reverse. Approval for one action is not approval for the next.

Requires asking first:
- Deleting or overwriting files you did not create in this session.
- Any database migration, schema change, or write against a non-local database.
- `rm -rf`, history rewrites, force pushes, or dropping a branch.
- Installing, upgrading, or removing dependencies.
- Anything that touches CI configuration, deployment, or infrastructure.
- Anything that sends data outward: opening a PR, posting to an API, sending a message.

Before overwriting or deleting anything, read it first. Describe what will change and wait for a clear yes. If the answer is ambiguous, it is a no.

Ask before any of these, which are breaking changes to a published contract:

- Adding an export to the barrel — it publishes API you must then support.
- Renaming a construct id, a public type, a prop, or a method.
- Changing the merge contract, or any documented default value.
- Widening or bumping a peer dependency range.
- Removing a deprecated alias.

- Never read `.env` files, credential stores, key material, or CI secrets unless the task cannot be done otherwise and the user has asked for it.
- Never print a secret's value — not in output, not in logs, not in a comment, not in an error message. Refer to it by name.
- Never paste a secret, token, internal hostname, or customer data into an external service, a commit, an issue, or a PR description.
- Never write a credential into source. Read it from the environment at runtime, and add a placeholder to the example env file instead.
- If you encounter a committed secret, stop and report it. Do not quote the value in the report.

## Reporting

Report outcomes faithfully. If tests fail, say so and show the output. If you skipped a
step, say which. If something is done and verified, say so plainly without hedging. Never
describe work as complete when part of it is unfinished — say what is left and why.
