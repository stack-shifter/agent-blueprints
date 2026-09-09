# TypeScript / Node — Agent Baseline

Rules for working in this repository. They apply to every file unless a more specific
instruction in the task overrides them.

## Project shape

A TypeScript service or library targeting Node. Source lives under `src/`, tests sit
beside the code they cover as `*.test.ts`. Compiled output is generated and never edited
by hand. Read the tree before assuming a layout beyond this.

## Toolchain

Use the package manager the lockfile indicates, and only that one. Do not mix.

- Install: `pnpm install` (or `npm ci` if the lockfile is `package-lock.json`)
- Typecheck: `pnpm typecheck` — must pass clean before any change is considered done
- Test: `pnpm test`
- Lint: `pnpm lint`
- Build: `pnpm build`

If a script is missing from `package.json`, say so rather than inventing an equivalent
command. Never add a dependency to solve a problem the standard library already solves.

## Strict typing

`strict: true` is assumed. Code that only compiles with it off does not ship.

- Never use `any`. When a type is genuinely unknown, use `unknown` and narrow it explicitly.
- Never use `as` to silence an error. A cast is a claim you cannot prove; fix the type instead. The only acceptable casts are on parsed external input immediately after validation.
- Never use non-null assertions (`!`). Narrow with a check, or make the type honest about nullability.
- No `@ts-ignore`. `@ts-expect-error` is permitted only with a comment naming the upstream issue it works around.
- Type the boundaries, infer the interior. Annotate exported signatures explicitly; let inference handle local variables.
- Prefer discriminated unions over optional-field soup. Make illegal states unrepresentable rather than checking for them.
- Validate all external input (network, filesystem, env, user) at the boundary with a schema, and type the interior from the schema's output.

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
- Match the surrounding file's existing vocabulary. If the codebase says `account`, do not introduce `user` for the same concept.

## Comments

Comment why, never what. If a comment restates the code, delete the comment or fix the name.

- Write a comment when the reason for the code is not recoverable from the code: a workaround, a non-obvious constraint, an ordering that matters, a deliberate deviation.
- Never leave commented-out code. Version control already remembers it.
- No changelog comments, no attribution comments, no `// TODO` without a concrete next action.
- Match the comment density of the surrounding file. A heavily annotated codebase and a bare one both have a house style; follow it.
- Doc comments on exported symbols state contract, not implementation: what it takes, what it returns, what it throws, what it assumes.

## Async and errors

- `await` every promise. A floating promise is a bug; never fire and forget without an explicit comment saying why.
- Use `Promise.all` for independent work rather than awaiting in sequence. Use a sequential loop only when order or backpressure matters.
- Throw `Error` subclasses, never strings or plain objects. Preserve the original with `{ cause }` when rewrapping.
- Catch narrowly. A `try` block wraps the one call that can fail, not the whole function body.
- Never swallow an error to make a path succeed. If it is genuinely ignorable, comment why.

## Testing

A bug fix starts with a test that reproduces the bug and fails. A feature starts with a test that describes the behavior.

- Write the failing test first, watch it fail for the right reason, then make it pass. A test that has never failed has not been verified.
- One behavior per test. The test name states the behavior in a sentence: `returns empty list when no matches`, not `search 2`.
- Assert on observable behavior, not on internal calls. A test that breaks on a refactor with no behavior change is a bad test.
- Cover the boundaries: empty, one, many, and the error path. The happy path alone is not coverage.
- Never weaken an assertion or delete a test to make a suite green. If a test is wrong, say so explicitly and explain why before changing it.
- Never report work as done on an unrun suite. If tests cannot be run, say that plainly rather than implying they passed.

Mock at the system's edges, nowhere else.

- Mock only what you do not own: network calls, clocks, randomness, the filesystem, third-party services.
- Never mock the thing under test, and never mock your own internal modules to make a test easier to write. That couples the test to the implementation and it will pass while the code is broken.
- Prefer a real in-memory implementation over a mock with scripted return values.
- If a test needs more than a couple of mocks to run, the design is telling you the unit has too many dependencies. Fix the design instead of adding mocks.
- Never assert that a mock was called as the primary assertion. Assert on the resulting state or output.

## Version control

Format: `type(scope): summary` — `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`.

- Summary in the imperative mood, lower case, no trailing period, under ~70 characters. "add retry to fetch client", not "Added retries."
- The summary says what changed. The body, when present, says why — the problem, the constraint, the alternative rejected. Never restate the diff in prose.
- One logical change per commit. If the summary needs "and", split the commit.
- Never mix a refactor with a behavior change in one commit. They need different review attention.
- Reference the issue in the body, not the summary.

Branch discipline:

- Never commit directly to the default branch. Branch first, always.
- Branch names: `type/short-description` in kebab-case — `fix/token-refresh-race`.
- Commit and push only when explicitly asked. Finishing a change is not permission to commit it.
- Never amend, rebase, force-push, or reset a branch that has been pushed, unless explicitly told to.
- Never use `git checkout .`, `git reset --hard`, or `git clean` to discard work you did not create. Uncommitted changes in the tree may be the user's.
- Never commit generated artifacts, dependency directories, or anything matched by the ignore file. Never commit a credential, even to a private repo.

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

Secrets:

- Never read `.env` files, credential stores, key material, or CI secrets unless the task cannot be done otherwise and the user has asked for it.
- Never print a secret's value — not in output, not in logs, not in a comment, not in an error message. Refer to it by name.
- Never paste a secret, token, internal hostname, or customer data into an external service, a commit, an issue, or a PR description.
- Never write a credential into source. Read it from the environment at runtime, and add a placeholder to the example env file instead.
- If you encounter a committed secret, stop and report it. Do not quote the value in the report.

## Reporting

Report outcomes faithfully. If tests fail, say so and show the output. If you skipped a
step, say which. If something is done and verified, say so plainly without hedging. Never
describe work as complete when part of it is unfinished — say what is left and why.
