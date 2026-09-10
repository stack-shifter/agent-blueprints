# Angular SPA — Agent Baseline

Rules for working in an Angular single-page application. They apply to every file unless a
more specific instruction in the task overrides them.

**Assumed majors:** Angular 21, TypeScript 5. Only a few rules below depend on a major, and
each says so where it does. If the project is on a different major, the project wins —
change this file rather than follow a rule that no longer applies.

## Project shape

Source lives under `src/app/`, bucketed by kind: components, pages, services, guards,
interceptors, directives, pipes, models. Keep the split between HTTP services and
everything else visible in the directory layout, so "where does an API call live" has one
answer.

The application bootstraps standalone — `bootstrapApplication` with a providers array,
no root module. Routes live in one exported `Routes` array. Read the tree before assuming
a layout beyond this.

## Toolchain

npm and the Angular CLI.

Read `package.json` and the workspace configuration for the repository's install,
development, build, test, lint, and format commands. If one is missing, report it rather
than composing an equivalent `ng` invocation by hand.

Run every configured check that applies; never infer that one script includes another.

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

Angular has a second layer of strictness that is separate from `strict` and off unless you
ask for it. Require an `angularCompilerOptions` block with `strictTemplates`,
`strictInjectionParameters`, and `strictInputAccessModifiers`. Without `strictTemplates`,
template type checking runs in a basic mode that will not catch a mistyped binding.

Two consequences worth stating outright, because both compile silently without strict null
checks and both lie about a type:

- **Declare inputs as `input.required<T>()` or `input<T>()` — never `input<T>(undefined)`.**
  The last form claims the signal holds a `T` while seeding it with `undefined`. Use
  `input.required<T>()` when the parent must supply it, and `input<T>()` when it is
  genuinely optional, which types it as `T | undefined` and forces you to handle that.
- Never pass a nullish initial value to a `nonNullable` form control.

A project adopting this baseline may start non-strict. Turning it on is the first task, not
a later cleanup, and the input declarations are where the errors will surface.

## Components

- `ChangeDetectionStrategy.OnPush` on every component. No exceptions — a single default
  component undermines the assumption everywhere else.
- Components are standalone. Never add a `standalone: true` key; it is the default and
  writing it is noise.
- Use the `input()` and `output()` signal functions. Never the `@Input` or `@Output`
  decorators.
- Bind host state through the `host` object in the decorator. Never `@HostBinding` or
  `@HostListener`.
- Always use `templateUrl`, never an inline `template`, regardless of how small the
  template is. Markup has one predictable home, tooling treats every template the same
  way, and a component that grows never needs its markup relocated mid-change.
- Keep a component focused. When one grows to mix data loading, modal orchestration, and
  connection lifecycle, extract rather than add another branch.

## Templates

- Use native control flow — `@if`, `@for`, `@switch`. Never the structural directives
  `*ngIf`, `*ngFor`, `*ngSwitch`.
- Always `track` a stable identity in `@for` — an id, or the value itself for a primitive
  list. Never `$index`.
- Use class and style bindings. Never `ngClass` or `ngStyle` — and do not route around the
  rule with inline `style` attributes or by setting styles imperatively from a directive
  when a class would do.
- Keep expressions simple. No arrow functions, no ternaries inside a `@for` expression, and
  no reliance on globals being available in template scope.
- Never call a method from a template to compute a display value. It re-runs on every
  change detection cycle; use a `computed()` signal instead.
- Read observables in the template with the `async` pipe. Do not `.subscribe()` in the
  class and assign to a field — the pipe subscribes and unsubscribes with the view, while
  a manual subscription is a teardown you have to remember.

## State and async

How server state is organised is a project decision. These are correctness rules that hold
whatever you choose.

- Derive with `computed()`. A value that can be derived is never stored and synced.
- **Never write a signal inside an `effect()`.** Reading one purely to register a
  dependency and then writing others is the pattern the framework warns against — express
  the relationship as a derived or linked signal instead.
- Never `.mutate()` a signal. Use `set()` or `update()`.
- Tear down with `takeUntilDestroyed(destroyRef)`. Do not hand-roll a destroy subject.
- Cancel every timer on destroy. A bare `setTimeout` that outlives its component fires
  against a dead view.
- **Assume an HTTP call may never emit at all.** An interceptor that short-circuits — for
  example returning an empty observable when an auth check fails — means neither the
  success nor the error handler runs, and a loading flag set beforehand stays true forever.
  Set loading state so that outcome is recoverable.

## Services and DI

- Inject with the `inject()` function. Never constructor parameter injection.
- Mark every injectable `providedIn: 'root'`. Do not also list it in a providers array —
  the redundancy invites a service that quietly depends on being listed there.
- Give HTTP error handling one home, so callers can branch on a known error shape. A new
  service that skips it makes every downstream error check silently fall through.
- Guards, interceptors, and resolvers are functional, not class-based.

## Routing

- Lazy-load every route with `loadComponent` or `loadChildren`. A statically imported page
  lands in the initial bundle.
- Give each route a distinct `title`. Routes sharing one literal string give no page
  context in browser history or to a screen reader.
- Prefer binding route parameters to component inputs over subscribing to the router's
  parameter observables.

## Forms

- Reactive forms only. Never `ngModel` or template-driven forms.
- Type the group — `FormGroup<T>` with an explicit control interface — so a renamed control
  is a compile error rather than a runtime undefined.
- Build the create and edit shapes from one definition. Duplicating every control across
  two branches of a switch is how the two drift apart.
- Guard against double submission: disable the form and the submit control while a
  submission is in flight.
- Derive error messages from the failing validator rather than hardcoding one message per
  field in the template.

## Accessibility

Every interface meets WCAG 2.2 AA. Accessibility is part of building the feature, not a pass that happens afterwards.

- Use the semantic element for the job. Something that performs an action is a button; something that navigates is a link. Never attach a click handler to a `div`, `span`, or icon element.
- Every interactive element is reachable and operable by keyboard in a logical order, with a visible focus indicator. Never remove a focus outline without replacing it with something at least as clear.
- Every control has an accessible name. An icon-only button needs an explicit label. Every image carries alt text, or empty alt when it is purely decorative.
- Associate each input with its label, mark invalid fields programmatically, and link the error message to the field it describes. An error conveyed only by styled text is invisible to a screen reader.
- A dialog identifies itself as a dialog, is labelled by its title, moves focus inward when it opens, keeps focus inside while it is open, closes on Escape, and returns focus to whatever opened it.
- Announce asynchronous outcomes — loading, success, failure — in a live region. A message that only appears visually is never heard.
- Meet AA contrast for text and for interactive controls. Never let colour alone carry meaning; pair it with text or a shape.
- Honour the reduced-motion preference for animation and transitions.
- Use real table markup for tabular data. `scope` belongs on a header cell, never on a data cell.

For Angular specifically:

- Enable the template accessibility lint ruleset and keep it passing. It catches the
  click-without-keyboard case automatically.
- **Never put `role="button"` and `tabindex="0"` on an element whose only handler is
  `(click)`.** It is announced as a button but cannot be activated by Enter or Space. Use a
  real `<button>`.
- **An `aria-label` overrides the visible name, so never paste a boilerplate one.** Example
  markup in component-library documentation often carries a label describing the demo
  rather than the action — copied into an application, a screen-reader user hears that
  description instead of what the control does. That is worse than no label at all.
- Manage focus with the framework's focus utilities, and announce asynchronous changes
  through a live announcer. Move focus when an inline editor opens and restore it on exit.

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

- Reach for `TestBed` only when the test needs dependency injection. Pure logic — a pipe, a
  validator, a mapper — is tested by calling it.
- Provide HTTP testing through the functional providers alongside the real HTTP provider.
  Never the legacy testing module.
- Call `verify()` on the HTTP testing controller in `afterEach`. An unasserted outstanding
  request is a test that did not test what it claims.
- Restore real timers in `afterEach` whenever a test faked them.
- Query rendered output by role and accessible name. A test that cannot find a control by
  its accessible name has found a real accessibility bug.

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

This stack builds and deploys a browser application. Ask before:

- Any deploy, bucket sync, or cache invalidation, in any environment.
- Adding, upgrading, or removing a dependency.
- Changing build budgets, browser targets, or the change-detection strategy of the app.

Stack-specific traps worth knowing before you touch them:

- **Enum values, not names, are the contract** wherever a magic string crosses a template
  boundary. Renaming a value silently breaks every call site the compiler cannot see.
- **Dates crossing an API that speaks date-only go through one conversion helper.** Parsing
  or formatting directly reintroduces the off-by-one-day bug the helper exists to prevent.
- **Do not assume zoneless change detection.** Check what the application actually provides
  before relying on zoneless semantics.
- Third-party configuration performed at module scope may run after bootstrap. Do not
  reorder it, and do not add initialization that depends on it having already run.

- Never read `.env` files, credential stores, key material, or CI secrets unless the task cannot be done otherwise and the user has asked for it.
- Never print a secret's value — not in output, not in logs, not in a comment, not in an error message. Refer to it by name.
- Never paste a secret, token, internal hostname, or customer data into an external service, a commit, an issue, or a PR description.
- Never write a credential into source. Read it from the environment at runtime, and add a placeholder to the example env file instead.
- If you encounter a committed secret, stop and report it. Do not quote the value in the report.

## Reporting

Report outcomes faithfully. If tests fail, say so and show the output. If you skipped a
step, say which. If something is done and verified, say so plainly without hedging. Never
describe work as complete when part of it is unfinished — say what is left and why.
