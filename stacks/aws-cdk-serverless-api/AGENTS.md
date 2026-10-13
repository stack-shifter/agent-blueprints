# AWS Serverless API (CDK + Lambda) — Agent Baseline

Rules for working in a production serverless API built with AWS CDK, API Gateway, and Lambda. The datastore may
be DynamoDB or a SQL database; the rules below say which parts depend on that choice. They apply to every file
unless a more specific instruction in the task overrides them.

This code runs in production and touches real infrastructure. Deployment is never a step you take on your own
initiative.

Read manifests, lockfiles, compiler configuration, and installed types for supported versions and interfaces.
Never infer them from another project.

## Project shape

Preserve the established architecture and test layout. The following describes preferred responsibilities; do
not migrate a working application solely to add folders or a dependency-injection layer. Keep CDK entrypoints
and stacks separate from runtime composition.

The application layers, and what each one is forbidden to do:

- `handlers/` — Lambda entry points and middleware composition only. No business logic, no SDK calls.
- `workers/` — scheduled/asynchronous entrypoints. Delegate to services; no business logic, SDK calls, or HTTP
  request middleware.
- `middlewares/` — reusable middleware. Short-circuit with a response. Replace request fields only with
  deliberately validated schema output.
- `controllers/` — extract parameters, orchestrate services and repositories, map errors to responses. Never
  call an AWS SDK client directly.
- `services/` — business logic and SDK calls. Throws domain errors. Knows nothing about HTTP.
- `data/` — repositories. Owns key construction and persistence. Wraps every failure in a domain error. Knows
  nothing about HTTP.
- `models/validation/` — schemas, one file per entity.

A layer may call downward, never upward. If a controller needs something from another controller, the logic
belongs in a service.

## Toolchain

npm and the AWS CDK CLI. Read `package.json` for the repository's build, test, diff, deploy, and
local-development scripts. Their names, account flags, profiles, and composed checks are project configuration.
If a script is missing, report it rather than composing an equivalent CDK command by hand.

Never deploy without being asked. Do not assume a command named "local" is a sandbox: inspect the script and
synthesized configuration, and treat access to imported or remote resources as a real external write.

## Infrastructure

Extend installed construct libraries or existing local constructs. Verify exports, defaults, merging, and
public-route overrides against their implementation. Never install a library or replace an API merely to match
an example.

- Centralize shared route settings and use explicit overrides. Verify whether null and undefined differ for
  authorizers; do not assume omission makes a route public.
- Grant each function only the resources and object prefixes it uses.
- Use supported package exports. Keep runtime imports separate from infrastructure imports; check transitive
  imports for accidental CDK inclusion.
- Use project response helpers consistently, preserving status, headers, CORS, problem reasons, and field-error
  contracts.

**Keep module-scope side effects out of anything `lib/` imports.** A stack that imports a handler module to
reach a name constant evaluates that whole import chain during `cdk synth`. An SDK client constructed at module
scope, or a required-environment assertion evaluated at import time, runs on every synth, diff, and deploy.

Configured handler names must match actual function exports. Use existing side-effect-free metadata or verified
strings; do not add a registry solely for this rule. Construct IDs must be unique. Explicit resource names must
distinguish deployment environments.

## Errors and validation

- Validate every request at the boundary with a schema. Keep schemas in `models/validation/`, one file per
  entity, each rule carrying an explicit human-readable message.
- Validation middleware owns parsed request input. Deliberately replace validated fields with schema output so
  coercions, defaults, and stripped values survive. Controllers may assert the parsed type only when configured
  validation runs first.
- Give JSON parsing one owner: validation or the existing body parser. Preserve parser errors and required
  authorization ordering; assertions do not parse input.
- **Every error response carries the CORS headers**, including responses constructed inside middleware. A
  validation failure that reaches a browser as a CORS error instead of a 400 is a bug, and it is invisible from
  the server side.
- **Route errors on typed error classes with `instanceof`.** Never branch on an error's `name` string: the class
  name and the assigned `name` drift apart silently, and every branch stops matching without a test failing.
- Catch as `unknown` and narrow before use. Keep `any` out of domain types.
- Services throw domain errors; controllers translate them to HTTP responses. Middleware may translate the
  boundary failures it owns.
- **Do not assert required configuration with `!`.** Read and validate required environment variables once at
  startup and fail loudly naming the variable that is missing. A non-null assertion turns a misconfiguration
  into a confusing runtime error.

## Data access

These hold whichever datastore you are on:

- Repositories own persistence. Query construction, keys, locks, and transactions live there — never in a
  controller.
- Keep HTTP and domain types independent of storage rows. A row shape is not an API contract, and letting one
  become the other couples your public surface to a schema change.
- Repositories map persistence failures to domain errors and never leak a raw driver or SDK exception upward.
- **Writes spanning two stores have no transaction.** When a write touches an external service and your
  datastore, write so a partial failure is recoverable: create the durable record first, or implement
  compensation. Never leave a record no route can reach.
- Right-size memory and timeout per route rather than accepting one default everywhere.

**When the datastore is DynamoDB.** Model keys and indexes around actual access patterns; preserve the existing
table design rather than forcing single-table architecture. Build keys in repositories, avoid unnecessary scans,
and grant required table/index access per function. Cursor pagination follows the query model.

**When the datastore is SQL.** Normalize, and let constraints do their job: primary and foreign keys, unique and
check constraints in the schema rather than only in application code. Add a named index per access pattern you
actually have, not per column you might filter on.

- Commit migrations and normally apply them through CI. Explicitly authorized manual execution requires
  verifying the exact target and scope. Never edit an applied migration; add a new one unless a disposable
  history rewrite is authorized.
- Transactions own atomicity. Use narrowly scoped locks for demonstrated integrity invariants and explain why.
  Document limits of application-only checks. Never hold transactions across external network calls.
- **Mind the connection budget.** Lambda scales horizontally and every concurrent execution wants its own
  connection, so concurrency multiplied by pool size can exhaust the server long before your traffic does. Put a
  pooler in front of the database, keep the per-function pool small, and treat a connection limit as a capacity
  number you reason about rather than discover in an incident.
- Configure VPC access only when database network topology requires it. Provide required routes/egress and
  credential access; public TLS endpoints do not automatically require a Lambda VPC.
- Add idempotency/replay storage or a transactional outbox for an explicit contract or demonstrated correctness
  need. Do not add them merely because retries exist. Keep required persistence in repositories.
- Numbered pagination is fine over SQL. Do not force a cursor model onto it because another service in the
  estate uses one.

**When using object storage.** Remove business rows first only when owed cleanup is recorded durably and
atomically. For expired uploads/exports, delete objects before removing tracking rows and retain failures for
retry. Handle partial batch failures, use persisted keys, and recheck pending-upload eligibility before storage
work. Database removal and lifecycle policies do not prove object deletion.

## Runtime layout

- Use named models/interfaces for object return types, including Promise results.
- Await into descriptive local variables before returning or constructing responses. Keep awaits inside their
  existing try/catch blocks to preserve rejection handling.
- In controllers, services, and repositories, order public operations Create → Read → Update → Delete when
  present; keep constructors first and helpers afterward.

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

Write comments that help a human follow the code: purpose, contracts, assumptions,
meaningful workflow stages, and rationale. Do not optimize for fewer comments.

- Use concise doc comments to explain a function's purpose and contract. Include
  the HTTP method and route for handlers or controllers where applicable.
- Use short inline comments to introduce meaningful logical steps, such as
  relying on validated input, delegating to a service, or protecting public error
  responses. Keep them even when a reader could infer the behavior from the code.
- Explain workarounds, non-obvious constraints, ordering requirements, and
  deliberate deviations from the normal pattern.
- Remove mechanical narration that adds no reading benefit, not useful summaries
  or workflow signposts. Correct inaccurate or stale comments when behavior changes.
- Never leave commented-out code. Version control already remembers it.
- No changelog comments, no attribution comments, no `// TODO` without a concrete
  next action or removal condition.
- Follow explicit project or user documentation preferences; otherwise match the
  closest existing peer in the same layer. Keep coverage uniform within a peer group.
- Document exported contracts: purpose, relevant inputs and outputs, assumptions,
  and meaningful errors. Use the language's documentation style without forcing
  redundant tags or narrating every statement.

## Deferred work

- Before completing a development cycle, triage every added or changed TODO/FIXME.
- Give temporary implementations a concrete replacement or removal trigger and
  track deferred work in the project's existing spec, plan, or work tracker.
- Resolve or update the TODO when a later cycle addresses the deferred work.

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

- Stub AWS SDK boundaries through existing seams rather than module interception.
- For SQL integration tests, use disposable databases with committed migrations and baseline fixtures. Mocked
  queries or another engine do not verify engine-specific constraints and transactions.
- Test middleware behavior directly against a constructed request object. Handler tests verify composed behavior
  through observable responses, not internal call counts.
- **Write template assertions for `lib/`.** Synthesize the stack and assert the resources it produces:
  permissions granted, environment variables set, authorizer attached. Stacks are code, and an unasserted stack
  is untested infrastructure.

- Template tests may disable bundling; separately verify real runtime bundles for entrypoints, accidental CDK
  inclusion, and size regressions.
- Local claim fixtures and synthesis do not verify deployed authorization. Report live authorization evidence
  separately.
- Test that composed validation passes coerced/defaulted/stripped values downstream and that parser and
  authorization failures retain their status and CORS headers.

## Test isolation

- Set deterministic test configuration before importing application modules. Never
  inherit developer service URLs, auth sessions, database URLs, or credentials.
- Stub external boundaries and block unexpected outbound calls, including failures
  caught by application code. Allow only explicitly configured disposable test
  services or deterministic stubs.
- Assert expected logs locally and fail on unexpected warnings/errors. Never mute
  the entire suite to hide missing fixtures or genuine failures.
- Keep automated tests independent of live services. Report explicitly authorized
  live verification separately; it does not replace deterministic tests.
- Follow existing test locations, naming, and shared setup. Do not reorganize tests
  or introduce another runner as incidental feature work.

## Verification

Run the project's checks before reporting a change complete. Never describe work as done on checks you did not run.

- The default gauntlet is typecheck, lint, test, build. Run all of it after any change to runtime code.
- Scale the checks to the work. A documentation-only or comment-only change does not need the full suite — say which checks you skipped and why.
- Run the checks the repository actually defines. If a script is missing, say so rather than substituting an equivalent command.
- Fix what the checks report before moving on. Never leave a failing check for the reviewer to find.
- If a check cannot run in this environment, name it and say so plainly. Do not imply it passed.
- Report the outcome, not the intent: name the commands you ran and what they returned.

For an infrastructure change, `cdk diff` is part of verification. Read the diff and report what it would change.
A diff showing a resource replacement is a finding, not a detail.

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

Explicit authorization covers the named scope and targets. Do not ask again for
ordinary edits or operations already approved. Read existing files first and
preserve unrelated work. Approval to implement phases does not authorize deployment
or writes against live data.

- Obtain explicit authorization for deployment/destruction, live-data writes,
  migration execution, dependency changes, history rewrites, force pushes,
  consequential deletion, external messages/PRs, or security-sensitive operations.
- Requests covering schema, infrastructure, CI, build, hosting, or migration
  authoring authorize those edits, not execution against a live target.
- Ask when target, risk, or scope materially expands. Ambiguous permission is not
  approval. Follow higher-priority per-action confirmation requirements.

Production datastores carry deletion protection and a retain policy. **Never lower either to make a deploy
succeed** — a deploy that requires it is a deploy that would destroy data. Stop and report what the diff shows
instead.

- Never read `.env` files, credential stores, key material, or CI secrets unless the task cannot be done otherwise and the user has asked for it.
- Never print a secret's value — not in output, not in logs, not in a comment, not in an error message. Refer to it by name.
- Never paste a secret, token, internal hostname, or customer data into an external service, a commit, an issue, or a PR description.
- Never write a credential into source. Read it from the environment at runtime, and add a placeholder to the example env file instead.
- If you encounter a committed secret, stop and report it. Do not quote the value in the report.

## Spec and plan requests

- Treat a request to create or persist a spec or plan as documentation scope.
  Write only the requested documents and explicitly requested related material.
- Do not implement feature phases until requested. Approval for multiple phases
  authorizes that range without repeated phase approval.
- Stop for failed gates, missing authority, or material scope changes. Keep
  execution evidence in the existing plan or tracking mechanism.
- Distinguish implemented behavior from deployment readiness that remains unverified.

## README Consistency Standard

Give readers a concise, familiar entry point. Standardize organization, not content,
tooling, or architecture; preserve useful repository-specific information.

### Recommended section order

Use these names and this order where applicable; omit irrelevant sections.

1. **Purpose** — Briefly explain what the repository does, who uses it, and its
   responsibility within the broader application.
2. **Architecture / Repository Overview** — Describe major components and connections;
   for docs repositories, describe organization and sources of truth. Include a small
   diagram only when it clarifies a relationship; accuracy matters more than format.
3. **Prerequisites and Configuration** — Identify tools, external resources, and required
   values before commands that need them. Separate local and deployment requirements;
   explain value sources, generated values, and public exposure. Never include secrets.
4. **Local Setup and Usage** — Give complete, ordered steps using actual package scripts,
   including tests/checks. For docs repositories, explain navigation, editing, preview,
   and validation instead of inventing application workflows.
5. **Deployment / Publishing** — Where applicable, separate Development and Production,
   first-time preparation and routine releases. Explain triggers, configuration,
   migration ordering, and publishing risks; link detailed runbooks.
6. **Teardown / Cleanup** — Show supported commands and exact targets where applicable.
   State what is deleted, retained, imported, or needs separate cleanup. Never imply
   stack destruction removes every related resource.
7. **Operational Notes and Limitations** — Include troubleshooting, compatibility, and
   deferred capabilities. Distinguish implemented behavior, known limitations, and
   checks or deployment behavior not yet verified.
8. **References** — Link useful specs, testing guides, contracts, and runbooks. Reuse
   existing indexes rather than adding exhaustive document catalogs.

### Content and editing rules

- Review the existing README and current source before reorganizing. Preserve important
  information; remove only duplication, obsolete guidance, or inaccuracies. When moving
  detail to a runbook, retain a short explanation and a working README link.
- Preserve API endpoints, authentication, configuration, database deployment,
  workers/retries, test prerequisites, and migration safety when applicable.
- Preserve UI routes, authentication/loading, environment variables, build labels,
  API connection requirements, and deployment details when applicable.
- Preserve docs requirements, prototypes, company standards, ownership, navigation,
  editing, and validation conventions. Put route/endpoint references near the overview
  or in a clearly named section nearby; do not force awkward placement.
- Keep commands copyable, correctly scoped, and consistent with scripts and pipelines.
  Do not impose another repository's framework, runner, environments, or architecture.
- Avoid completed-cycle handoffs, repeated spec narratives, and speculative features
  presented as current behavior. README refresh work does not authorize runtime code,
  infrastructure, dependency, or deployment changes.

### Verification

Verify commands, routes, configuration, links, and operational claims against current
source. Run applicable Markdown formatting and whitespace checks. Report unverified
statements plainly; local tests alone do not establish production readiness.

## Reporting

Report outcomes faithfully. If tests fail, say so and show the output. If you skipped a step, say which. If
something is done and verified, say so plainly without hedging. Never describe work as complete when part of it
is unfinished — say what is left and why.
