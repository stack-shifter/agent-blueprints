# AWS Serverless API (CDK + Lambda) — Agent Baseline

Rules for working in a production serverless API built with AWS CDK, API Gateway, and
Lambda. The datastore may be DynamoDB or a SQL database; the rules below say which parts
depend on that choice. They apply to every file unless a more specific instruction in the
task overrides them.

This code runs in production and touches real infrastructure. Deployment is never a step
you take on your own initiative.

**Assumed majors:** Node 24, TypeScript 5, AWS CDK 2. Only a few rules below
depend on a major, and each says so where it does. If the project is on a different major,
the project wins — change this file rather than follow a rule that no longer applies.

## Project shape

`bin/` holds the CDK app entry point. `lib/` holds the stacks. `src/` holds the
application, layered. Follow the project's established test layout.

The application layers, and what each one is forbidden to do:

- `handlers/` — Lambda entry points and middleware composition only. No business logic,
  no SDK calls.
- `middlewares/` — reusable middleware. Short-circuit by returning a response. Never
  mutate the event.
- `controllers/` — extract parameters, orchestrate services and repositories, map errors
  to responses. Never call an AWS SDK client directly.
- `services/` — business logic and SDK calls. Throws domain errors. Knows nothing about
  HTTP.
- `data/` — repositories. Owns key construction and persistence. Wraps every failure in a
  domain error. Knows nothing about HTTP.
- `models/validation/` — schemas, one file per entity.

A layer may call downward, never upward. If a controller needs something from another
controller, the logic belongs in a service.

## Toolchain

npm and the AWS CDK CLI. Read `package.json` for the repository's build, test, diff,
deploy, and local-development scripts. Their names, account flags, profiles, and composed
checks are project configuration. If a script is missing, report it rather than composing
an equivalent CDK command by hand.

Never deploy without being asked. Do not assume a command named "local" is a sandbox:
inspect the script and synthesized configuration, and treat access to imported or remote
resources as a real external write.

## Infrastructure

Build on the shared construct library rather than hand-rolling API Gateway, Lambda
integrations, and IAM wiring. Import it from the package root; deep imports are blocked.

Compose routes with the `RestApi` construct. Its props take `name`, `corsOrigins` as an
array, and `defaultRouteOptions` for settings shared by every route. Its route methods are
`get`, `post`, `put`, and `delete` — there is no separate by-id method, so a path
parameter route is a `get` with the parameter in `routePath`.

Each route's options take `functionName`, `entry`, `handler`, and `routePath`, plus
`grants`, `scopes`, `requestParameters`, and `model` as needed. `routePath` must start
with `/`.

- Shared settings go in `defaultRouteOptions` once. Per-route options merge over them:
  scalars overwrite, `environment` and `bundling` deep-merge, and `grants` are added to
  the defaults rather than replacing them.
- Set `inheritDefaults: false` to opt a route out of defaults entirely.
- **To make one route public, pass `authorizer: null`.** Passing `undefined` means
  "inherit the default", which leaves the route authorized. These are different values and
  the distinction is deliberate.
- Grant IAM with callbacks — `grants: [(fn) => table.grantReadData(fn)]` for a table,
  `grants: [(fn) => secret.grantRead(fn)]` for a database credential. Grant each function
  only the resources it uses. Never attach a broad policy to cover several routes.
- Import existing infrastructure through the `Importer` helpers rather than scattering
  `from*` lookups across the stack.
- Use `LambdaNode` for a standalone function that is not an API route.

Return responses through the framework's `RestResult` helpers — `ok`, `created`,
`noContent`, `badRequest`, `notFound`, `forbidden`, `unauthorized`,
`internalServerError`, `problem`. They are lower camelCase. Never hand-build a response
object; that is how a header or a status code drifts.

**Keep module-scope side effects out of anything `lib/` imports.** A stack that imports a
handler module to reach a name constant evaluates that whole import chain during `cdk
synth`. An SDK client constructed at module scope, or a required-environment assertion
evaluated at import time, runs on every synth, diff, and deploy.

**A handler's name string doubles as its construct id.** It must match the exported
handler function's name so Lambda can find it, and it must be unique within the stack so
two routes do not collide. Keep the names in one exported constant map and assert them in
tests.

## Errors and validation

- Validate every request at the boundary with a schema. Keep schemas in
  `models/validation/`, one file per entity, each rule carrying an explicit human-readable
  message.
- **Validation middleware validates; it does not transform.** Never assume a controller
  receives coerced, defaulted, or stripped input — a numeric query parameter is still a
  string. If you need the parsed value downstream, write it back onto the event
  deliberately and say so.
- **Every error response carries the CORS headers**, including responses constructed
  inside middleware. A validation failure that reaches a browser as a CORS error instead
  of a 400 is a bug, and it is invisible from the server side.
- **Route errors on typed error classes with `instanceof`.** Never branch on an error's
  `name` string: the class name and the assigned `name` drift apart silently, and every
  branch stops matching without a test failing.
- Catch as `unknown` and narrow before use. Keep `any` out of domain types.
- Services throw domain errors. Controllers are the only layer that turns an error into a
  status code.
- **Do not assert required configuration with `!`.** Read and validate required
  environment variables once at startup and fail loudly naming the variable that is
  missing. A non-null assertion turns a misconfiguration into a confusing runtime error.

## Data access

These hold whichever datastore you are on:

- Repositories own persistence. Query construction, keys, locks, and transactions live
  there — never in a controller.
- Keep HTTP and domain types independent of storage rows. A row shape is not an API
  contract, and letting one become the other couples your public surface to a schema
  change.
- Repositories map persistence failures to domain errors and never leak a raw driver or
  SDK exception upward.
- **Writes spanning two stores have no transaction.** When a write touches an external
  service and your datastore, write so a partial failure is recoverable: create the
  durable record first, or implement compensation. Never leave a record no route can
  reach.
- Right-size memory and timeout per route rather than accepting one default everywhere.

**When the datastore is DynamoDB.** Single-table with a partition key and sort key. Add a
global secondary index for an alternate lookup rather than scanning. Build key strings in
one place in the repository. Grant table and index access per function with grant
callbacks. Cursor pagination follows from the query model — do not fake page numbers over
it.

**When the datastore is SQL.** Normalize, and let constraints do their job: primary and
foreign keys, unique and check constraints in the schema rather than only in application
code. Add a named index per access pattern you actually have, not per column you might
filter on.

- **Migrations are committed to the repository and applied by CI.** Never run a migration
  by hand against a deployed database, and never edit a migration that has already been
  applied — add a new one.
- Transactions own atomicity; take row locks inside the repository that needs them. Keep
  a transaction as short as the invariant requires and never hold one across a network
  call to another service.
- **Mind the connection budget.** Lambda scales horizontally and every concurrent
  execution wants its own connection, so concurrency multiplied by pool size can exhaust
  the server long before your traffic does. Put a pooler in front of the database, keep
  the per-function pool small, and treat a connection limit as a capacity number you
  reason about rather than discover in an incident.
- A function that reaches a database in a VPC must be placed in that VPC with egress, and
  granted the credential it reads. That placement changes cold starts — do it
  deliberately, not by default for every route.
- Use an idempotency key for any operation a client may retry, and a transactional outbox
  when a write must also produce an event. Both belong in the repository layer.
- Numbered pagination is fine over SQL. Do not force a cursor model onto it because
  another service in the estate uses one.

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

For this stack specifically:

- Mock at the AWS SDK boundary and nowhere else. Inject a stub document client into a
  repository rather than intercepting the module.
- Test middleware behavior directly against a constructed request object. Handler tests
  verify composition — that the middleware is wired and the controller is called — not
  middleware behavior.
- **Write template assertions for `lib/`.** Synthesize the stack and assert the resources
  it produces: permissions granted, environment variables set, authorizer attached. Stacks
  are code, and an unasserted stack is untested infrastructure.

## Verification

Run the project's checks before reporting a change complete. Never describe work as done on checks you did not run.

- The default gauntlet is typecheck, lint, test, build. Run all of it after any change to runtime code.
- Scale the checks to the work. A documentation-only or comment-only change does not need the full suite — say which checks you skipped and why.
- Run the checks the repository actually defines. If a script is missing, say so rather than substituting an equivalent command.
- Fix what the checks report before moving on. Never leave a failing check for the reviewer to find.
- If a check cannot run in this environment, name it and say so plainly. Do not imply it passed.
- Report the outcome, not the intent: name the commands you ran and what they returned.

For an infrastructure change, `cdk diff` is part of verification. Read the diff and report
what it would change. A diff showing a resource replacement is a finding, not a detail.

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

This stack deploys real infrastructure. Ask before:

- Any `cdk deploy`, `cdk destroy`, or data migration script, in any environment.
- Changing a table's schema, keys, indexes, removal policy, or deletion protection.
- Writing or applying a database migration.
- Anything touching the AWS profile, the CI role, or deployment workflows.

Production datastores carry deletion protection and a retain policy. **Never lower either
to make a deploy succeed** — a deploy that requires it is a deploy that would destroy
data. Stop and report what the diff shows instead.

- Never read `.env` files, credential stores, key material, or CI secrets unless the task cannot be done otherwise and the user has asked for it.
- Never print a secret's value — not in output, not in logs, not in a comment, not in an error message. Refer to it by name.
- Never paste a secret, token, internal hostname, or customer data into an external service, a commit, an issue, or a PR description.
- Never write a credential into source. Read it from the environment at runtime, and add a placeholder to the example env file instead.
- If you encounter a committed secret, stop and report it. Do not quote the value in the report.

## Reporting

Report outcomes faithfully. If tests fail, say so and show the output. If you skipped a
step, say which. If something is done and verified, say so plainly without hedging. Never
describe work as complete when part of it is unfinished — say what is left and why.
