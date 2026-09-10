# ASP.NET Core REST API (C#) — Agent Baseline

Rules for working in a layered ASP.NET Core REST API written in C#, backed by Postgres
through Entity Framework Core and fronted by JWT bearer authorization. They apply to every
file unless a more specific instruction in the task overrides them.

**Assumed majors:** .NET 10, C# 14, ASP.NET Core 10, EF Core 10, Npgsql 10. Only a
few rules below depend on a major, and each says so where it does. If the project is on a
different major, the project wins — change this file rather than follow a rule that no
longer applies.

## Project shape

Read the solution before assuming project boundaries or test layout.
The layers, and what each one is forbidden to do:

- `Program.cs` — the composition root. Registers services, configures the pipeline, maps
  controllers. No business logic.
- `Controllers/` — HTTP only. Read the request, call a repository or service, shape the
  response. Never touch the `DbContext`, a `DbSet`, or an `IQueryable`.
- `Services/` — business logic and third-party integrations, one concern each behind an
  interface. Throws domain exceptions; knows nothing about HTTP.
- `Repositories/` — owns every query. Wraps failures in a repository exception. Knows
  nothing about HTTP.
- `Repositories/Configuration/` — one `IEntityTypeConfiguration<T>` per entity.
- `Models/` — `Entities/`, `DTOs/`, `QueryParams/`, `Enum/`. `Migrations/` is generated
  and never hand-written.

A layer may call downward, never upward. The aggregate repository interface is the only
data-access seam a controller may hold; a controller that injects the `DbContext` directly
is a layering break, not a shortcut.

## Toolchain

Use the .NET SDK selected by the repository and its configured integration-test runtime.

- Restore, build, test: `dotnet restore`, `dotnet build`, `dotnet test`
- Run locally: `dotnet run --project <ApiProject>`
- Migrations: `dotnet ef migrations add <Name> --project <ApiProject>`, then
  `dotnet ef database update --project <ApiProject>`

Use the repository-defined command for its local database and other integration-test
prerequisites; do not assume Docker Compose is present.

Read the solution and `global.json` for real project names and the SDK band rather than
assuming them. **`dotnet ef` needs `--project`** unless you are inside the API project.

Read the test configuration before assuming integration tests use Docker or a particular
database. If an external prerequisite is unavailable, report that rather than calling the
suite broken.

## Configuration

Configuration comes from `appsettings.json`, environment-specific overlays, and environment
variables. Secrets come from the environment only.

- Every setting the code reads is present in `appsettings.json` with a safe placeholder, so
  the required set is discoverable without reading the source.
- Bind configuration into typed options and inject those. Never call
  `Environment.GetEnvironmentVariable` from inside a service — it hides a dependency the
  constructor should declare and cannot be substituted in a test.
- **No endpoint, identifier, or origin is hardcoded in `Program.cs`.** Authority URLs,
  token issuers, allowed CORS origins, and bucket names are configuration. A literal in the
  composition root means every environment ships the same one — including the wrong one.

## Controllers

One controller per resource, attribute-routed under a version prefix, `[ApiController]`,
`[Authorize]` at the class level, dependencies through a primary constructor.

- Return `ActionResult<T>` when there is a body and `IActionResult` when there is not, and
  use the helpers — `Ok`, `CreatedAtAction`, `NoContent`, `NotFound`, `BadRequest` —
  rather than constructing status codes by hand. `[ApiController]` already returns 400
  with a problem payload for an invalid model; never hand-roll that check.
- Keep actions thin: validate what the attributes cannot, call one repository or service,
  map, return. Business rules belong in a service.
- **An empty collection is `200` with an empty page, never `404`.** A filter that matched
  nothing is a successful query. Reserve `404` for a single resource that does not exist.
- `POST` returns `201` through `CreatedAtAction` naming the read action, so the `Location`
  header is generated rather than typed.
- `PUT` and `DELETE` return `204`, and a route id that disagrees with a body id is a `400`
  before any work happens.
- Accept a `CancellationToken` and pass it down; a disconnected client should not leave a
  query running.

## Errors

Repositories and services throw typed exceptions; controllers translate them into
responses. The exception types are the contract between the layers.

- Catch the specific exception types you can map. Never `catch (Exception)` in a
  controller — an unexpected failure must not be reported as a handled one.
- **The message returned to the caller is fixed text, never the exception's message.**
  Log the exception with its stack; return wording that says what failed without naming a
  table, a query, or a file path.
- One wording per operation, reused across every controller. Drift between "database" and
  "datastore" makes the response text untestable and the logs unsearchable.
- Log through the injected logger, passing the exception so the stack is preserved. Never
  `Console.WriteLine`. A repository exception is not an HTTP status: it carries no status
  code and knows none.

## Validation

Request validation is declarative, on the Post and Put DTOs, using data annotations.

- Use the built-in `RequiredAttribute` from `System.ComponentModel.DataAnnotations`.
  ASP.NET Core model validation recognizes attributes derived from `ValidationAttribute`;
  an unrelated, identically named attribute is only metadata and is not enforced. Check the
  resolved attribute type whenever a validation rule appears not to fire.
- Constrain every string with a length, every number with a range, and every enum with an
  explicit enum check. Supply the message where the default would not tell a caller what to
  send.
- The Put DTO inherits the Post DTO and adds only identity and update-only fields.
- Query parameters are `record` types deriving from a shared base carrying paging and
  sort. **Never shadow a base member with `new`** — the shadow drops the base member's
  validation attributes, and anything holding the base type reads the old default.

## Data access

One repository per aggregate behind an interface, all of them exposed as properties on a
single aggregate interface that controllers inject. Repositories take the `DbContext`
through a primary constructor.

- Keep the contract small and honest: get by id, list with a filter, add, update, delete.
  **Do not put a method on the interface that every implementation throws
  `NotImplementedException` for** — delete it, and add it back when something needs it.
- Read queries start with `AsNoTracking()`, once, at the top. Applying it twice is noise
  that reads like a second opinion.
- Consider `AsSplitQuery()` when sibling collection includes would multiply rows. Do not
  apply it mechanically: split queries add round trips and can observe inconsistent data.
- Fetch a single entity for reading with `AsNoTracking()`, the includes it needs, and
  `FirstOrDefaultAsync`. Use `FindAsync` only when the entity is about to be mutated,
  because it returns a tracked instance. `SingleOrDefaultAsync` on a primary key adds
  duplicate-detection semantics that an enforced unique key already provides.
- Update by loading the tracked entity, returning early when it is missing, assigning only
  the mutable fields, stamping the modified time, and saving. Never copy the id or created
  time from the incoming object, and return the tracked entity, not the argument.
- Delete is idempotent: load, return when absent, remove, save.
- Wrap failures in the repository's exception type with the original as the inner
  exception. Wrap every public method the same way — one unwrapped method leaks a provider
  exception into a controller that is not expecting it.
- Never let `IQueryable` escape a repository; materialize before returning. Never build
  SQL by concatenation — parameterize, or use the query API.

## Pagination and ordering

Paging is offset-based: a page number and a page size in, items plus totals out. The
envelope type initializes its collection so it is never null.

- Apply the filter, then the ordering, then the page. Count with a separate query against
  the same filter.
- **Order by a key set that ends in the primary key, always.** Sorting on a name, a status,
  or a date alone is not a total order: rows with equal values can land on either side of a
  page boundary, so the same row appears twice or disappears entirely as the caller pages.
  This is the most common and least visible paging defect.
- The tie-breaker runs in the same direction as the primary sort. A descending sort with an
  ascending tie-breaker orders rows differently than anyone reading the code expects.
- The page size has a maximum, enforced by the query type. Never remove the cap to satisfy
  a caller who wants every row; give them an export endpoint instead.

## Schema and migrations

One `IEntityTypeConfiguration<T>` per entity, applied explicitly, with the table and every
column named in full.

- Name the table and every column rather than relying on convention, so a rename in C# does
  not silently become a migration.
- Store enums as strings with an explicit two-way conversion and a length. An enum stored
  by ordinal breaks the moment someone inserts a member.
- Give money and quantity columns an explicit precision and scale. A default-precision
  decimal will round in a way nobody chose.
- Declare each relationship once, on one side, with an explicit foreign key and delete
  behavior. Two half-declarations is how a cascade gets lost.
- Seed data sits behind an environment check and never runs outside development.

Migrations are generated, reviewed, and committed with the schema change that produced
them.

- Read the generated migration before committing it — a column rename that generates as a
  drop and an add will destroy data.
- **Never edit a migration that has been applied anywhere.** Write a new one. Never change
  an entity or its configuration without generating the migration in the same change.

## Models and nullability

Entities are persisted shapes, DTOs are wire shapes, and neither is ever the other. A
controller returns DTOs; a repository returns entities.

- Entities carry identity and audit fields; DTOs carry what the caller should see. Never
  return an entity from an action — it leaks navigations and audit columns by accident.
- Times are stored and returned in UTC, and business dates use a date-only type so no
  timezone can shift them. Convert for display only, never on the way into storage.

Nullable reference types are enabled, and the compiler's warnings are the point of the
feature.

- **Never use the null-forgiving operator (`!`) to silence a warning**, and never quiet a
  required navigation with `null!` or `default!`. Narrow with a check, or make the type
  honest: either the member is optional and says so, or it is `required`. Suppress a
  nullable warning only with a comment naming the invariant that makes it safe.
- Never assume a navigation is populated unless the query or explicit loading established
  it. Relationship fix-up or lazy loading may populate it in some context states and not
  others, so a passing tracked query does not prove the include is unnecessary.
- Non-nullable strings get an initializer; optional ones are nullable and get none. A
  non-nullable collection is initialized empty, never left null.

## Services

A service owns one integration or one piece of business logic, behind an interface, and is
registered in the composition root.

- **Mapping is pure.** A mapper translates between shapes and does nothing else — no I/O,
  no network calls, no signing, no database access. A mapper that calls a remote service
  runs once per item, so mapping a page of results silently multiplies that call by the
  page size. Do that work once in the caller and pass the result in.
- Take collaborators through the constructor. A service that reaches for ambient state
  cannot be tested without that state.
- **One clock, injected, computed on read.** Never call `DateTime.Now` — it is the server's
  local time and it will differ between a developer's machine and production. Never call
  `DateTime.UtcNow` directly in code you want to test. A clock that captures the time in a
  field initializer is frozen for the lifetime of its scope; expose it as a property that
  reads the current time. One clock per codebase — if one already exists, do not add a
  second on a different interface.

## Storage and uploads

Uploads are the largest untrusted input the API accepts, and the object store will store
whatever it is handed.

- **Validate content type, extension, and size before the stream reaches the object
  store.** A request size limit on the action bounds the request; it does not tell you the
  bytes are what the caller claimed. Check the declared type against an allow-list, never a
  deny-list.
- Never build a storage key from a caller-supplied filename. Generate it, and keep the
  original name as metadata to restore on download.
- Signed URLs get the shortest lifetime that works, taken from configuration. Dispose
  streams and requests; a path that leaks one per upload holds memory until collection.
- Wrap failures in the storage exception type with the original as the inner exception, and
  never return the provider's message to the caller.

## Naming

- Names state what a thing is or does, not how it is implemented. `activeDrivers`, not
  `filteredList`.
- Follow the language's casing without exception: PascalCase for types, methods,
  properties, and constants; camelCase for locals and parameters; an `I` prefix on
  interfaces. Async methods end in `Async`.
- Booleans read as assertions: `IsReady`, `HasAccess`, `ShouldRetry`. Never negated in the
  name — `IsEnabled` over `IsNotDisabled`.
- No abbreviations except ones universal in the domain (`id`, `url`, `db`). Name length
  scales with scope: a loop index may be `i`; a public member may not.
- One public type per file, and the file takes that type's name. Types carry their layer as
  a suffix — the controller, repository, service, mapper, and configuration for a resource
  all read as such from the file list alone.
- Namespaces match folders. A file whose namespace disagrees with its directory is a file
  that was moved without being read.
- Match the surrounding vocabulary. If the codebase says `driver`, do not introduce `user`
  for the same concept.

## Prefer boring code

When several implementations work, choose the one a mid-level developer understands on the first read. Cleverness is a cost paid by every future reader.

- Prefer plain control flow — `if`, `for`, early returns — over deeply chained pipelines or condensed one-liners. Favor code that can be stepped through in a debugger.
- Follow the patterns already in the repository. Do not introduce a second architectural style alongside a working one.
- Some duplication is acceptable when it keeps logic readable and local. Do not abstract on the second occurrence; wait for the third.
- Build only what the current task requires. Never generalize for a hypothetical future caller.
- Do not introduce factories, builders, dependency-injection layers, or similar patterns unless the problem clearly demands one.
- Do not add a dependency for something the standard library already does.
- Added complexity is fine when it buys correctness, performance, or maintainability — but comment why it was worth the cost.

## Vertical code layout

Write code that reads top to bottom like prose. Density is not concision.

- One statement per line. Never chain unrelated operations onto one line to save space.
- Separate logical stanzas with a single blank line. Group a variable's declaration with the lines that first use it.
- Declare variables close to first use, not in a block at the top of the function.
- Put the early returns and guard clauses first, then the main path. Never wrap the main path in an `else`.
- Order functions so a caller appears above the functions it calls, so the file reads at descending levels of detail.
- Break a long expression across lines at the operators, aligning the operands — do not let it wrap arbitrarily at the margin.
- Keep functions short enough to see whole. If you cannot see the top and bottom at once, extract a named helper.

## Comments

Comment why, never what. If a comment restates the code, delete the comment or fix the name.

- Write a comment when the reason for the code is not recoverable from the code: a workaround, a non-obvious constraint, an ordering that matters, a deliberate deviation.
- Never leave commented-out code. Version control already remembers it.
- No changelog comments, no attribution comments, no `// TODO` without a concrete next action.
- Do not decide comment style per file. Match the closest existing peer in the same layer — a heavily annotated codebase and a bare one both have a house style.
- Keep coverage uniform within a peer group. Documenting some exported functions in a module while leaving equivalent ones bare is worse than documenting none.
- Doc comments on exported symbols state contract, not implementation: what it takes, what it returns, what it throws, what it assumes.

The peer groups here are the layer folders, and the floor in each is an XML doc comment on
every public type and member. Actions name the method and route they serve; methods that
throw document what they throw. Inline comments mark non-obvious stages only.

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

- Use the project's configured test runner and established test layout.
- Mock the aggregate repository interface and the third-party SDK boundary. **Use the real
  mapper** — it is pure, and substituting it means the mapping is never actually tested.
- Never assert only that a mock was called. Assert the returned result type, the status
  code, and the payload. Verifying interactions is a supplement to that, not a substitute.
- Do not use a mock setup with no return value as a way to produce null; return it
  explicitly, or the next reader cannot tell the stub from an oversight.
- Repository integration tests leave their database as they found it. Use the cleanup
  mechanism established by the fixture; a test that mutates shared seeded rows makes later
  tests order-dependent.
- When the integration suite is meant to validate migrations, apply migrations in the
  fixture rather than creating the schema from the model.
- Never hardcode a generated identifier from seeded data — query for the row you need, or
  insert your own. Cover the authorization and validation failures, not only success.

## Verification

Run the project's checks before reporting a change complete. Never describe work as done on checks you did not run.

- The default gauntlet is typecheck, lint, test, build. Run all of it after any change to runtime code.
- Scale the checks to the work. A documentation-only or comment-only change does not need the full suite — say which checks you skipped and why.
- Run the checks the repository actually defines. If a script is missing, say so rather than substituting an equivalent command.
- Fix what the checks report before moving on. Never leave a failing check for the reviewer to find.
- If a check cannot run in this environment, name it and say so plainly. Do not imply it passed.
- Report the outcome, not the intent: name the commands you ran and what they returned.

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

This API owns a real database and real cloud resources. Ask before:

- Applying a migration, running a seed, or writing to any database you did not create for
  this task.
- Editing or deleting anything under the migrations folder.
- Changing a CI workflow, the container image definition, or the compose file; adding,
  upgrading, or removing a package; changing the target framework or SDK band.
- Any real call to the object store, the mail service, or another cloud service.

- Never read `.env` files, credential stores, key material, or CI secrets unless the task cannot be done otherwise and the user has asked for it.
- Never print a secret's value — not in output, not in logs, not in a comment, not in an error message. Refer to it by name.
- Never paste a secret, token, internal hostname, or customer data into an external service, a commit, an issue, or a PR description.
- Never write a credential into source. Read it from the environment at runtime, and add a placeholder to the example env file instead.
- If you encounter a committed secret, stop and report it. Do not quote the value in the report.

## Reporting

Report outcomes faithfully. If tests fail, say so and show the output. If you skipped a
step, say which. If something is done and verified, say so plainly without hedging. Never
describe work as complete when part of it is unfinished — say what is left and why.
