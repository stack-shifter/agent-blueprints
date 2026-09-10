# React SPA (Vite, full stack) — Agent Baseline

Rules for working in a React single-page application built with Vite and TypeScript,
using Cognito hosted-UI auth, Bootstrap, and React Hook Form with Zod. They apply to
every file unless a more specific instruction in the task overrides them.

**Assumed majors:** React 19, React Router 8, Vite 7, TypeScript 5. Only a few rules below
depend on a major, and each says so where it does. If the project is on a different major,
the project wins — change this file rather than follow a rule that no longer applies.

## Project shape

A single-page application. Source lives under `src/`, grouped by purpose rather than by
feature: pages, reusable pieces of interface, hooks, contexts, models, and utilities each
get a directory. The entry point mounts the app and installs whatever providers wrap it.

Static assets live in the public directory. Build output is generated and never edited by
hand. Read the tree before assuming a layout beyond this.

## Toolchain

npm, Vite, and TypeScript.

Read `package.json` for the repository's install, development, build, lint, format, test,
and typecheck commands. Script names and composition are project configuration. If a
script is missing, report it rather than inventing an equivalent command.

The project's formatter configuration is authoritative. Read it and run the configured
script; never duplicate its indentation, width, quote, or comma settings in prose.

Read `package.json` to learn which checks each script composes. Run every configured check
that applies; never infer that one command includes another.

## Components

- Function components only. Type props as `<Name>Props` and destructure them in the
  signature.
- One component per file, named for the file. Components are `PascalCase.tsx`; everything
  else uses a dotted suffix — `user.service.ts`, `auth.hook.ts`, `user.model.ts`.
- Co-locate a component's stylesheet beside it and import it from the component file.
- No barrel files. Import from the defining module directly.
- Keep a component small enough to read whole. When it grows past that, extract the
  presentational part rather than adding another conditional branch.

## Routing

Use the data router.

- Import `createBrowserRouter` and `RouterProvider` from `react-router/dom`. Everything
  else — `Link`, `NavLink`, `Outlet`, `Navigate`, `useNavigate`, `useParams`,
  `useLoaderData`, `useRouteError` — comes from `react-router`. **React Router 8 requires
  this split.** Version 7 supported `react-router/dom` while retaining compatibility
  exports; check which major the project has before moving an import.
- **The `react-router-dom` package no longer exists.** Never install it and never import
  from it.
- Middleware is the default and the future flags that once gated it have been removed.
  Do not add them back.
- In `meta` and `matches`, the field holding a route's loaded data is `loaderData`, not
  `data`.
- Define routes with `Component:` and lazy-load route modules. Importing every page
  statically puts the whole application in the initial bundle.
- Give the router an error boundary. Narrow the thrown value before reading it — an
  unchecked cast of a routing error is a crash inside the error handler.

## Auth

Cognito hosted UI through Amplify.

- Configure Amplify once, at the entry point, before anything reads a session.
- Retrieve the session and access token through one shared accessor. Token refresh is the
  library's job — never hand-roll expiry checks or a refresh interceptor.
- Centralise group and role checks in one helper. The same membership comparison inlined
  across several places is how one of them silently gets it wrong.
- Authorization in the browser is presentation, not enforcement. Hiding a control does not
  protect the operation behind it; the API must enforce it too.
- **If the auth guard wraps the router, no route can be public.** Adding a public route is
  a restructure of the entry point, not a change to the route table. Decide this before
  building a login or marketing page.

## Forms and validation

React Hook Form with a Zod resolver.

- Inputs are uncontrolled, registered with `register()`. Reach for `Controller` only when
  a control cannot forward a ref.
- Schemas live in one validation directory, one file per entity. Every rule carries an
  explicit, human-readable message. Export the inferred type alongside the schema rather
  than restating the shape by hand.
- Always supply `defaultValues`. An input that starts undefined and later receives a value
  switches from uncontrolled to controlled and warns.
- Disable the form while a submit is in flight, and show that state on the submit control.
- Map server-side failures back onto the fields that caused them. A validation error that
  only appears as a toast leaves the user hunting for the bad field.
- Transform values at the boundary — parse on the way in, format on the way out. Do not
  mutate the submitted object in the submit handler.

## Styling

Bootstrap loaded through SCSS.

- One co-located stylesheet per component. Namespace class names
  `block__element--modifier` so a global stylesheet behaves like a scoped one.
- Reserve the global stylesheet for design tokens, resets, and genuinely shared utilities.
  A one-off rule for a single component does not belong there.
- Prefer the framework's utility classes over a bespoke rule that duplicates one.
- **Define each breakpoint once.** A pixel value duplicated between a media query and a
  measuring hook will drift, and the two will disagree about what "mobile" means.
- Import a stylesheet you depend on. Applying classes from a library that was never
  imported produces markup that silently does nothing.

## Effects and state

These are correctness rules. How you organise data fetching is a project decision; that
these hold is not.

- **Never call a state setter during render.** A setter reached during render re-runs on
  every render and can loop indefinitely. Side effects belong in an effect or an event
  handler.
- **After `await`, never read the state variable you expected the call to set.** It still
  holds the value captured when that render ran. Wrap the awaited call in `try`/`catch`
  and branch on the caught error instead. Reading the stale variable is how a failed
  operation reports success.
- **Every effect that starts something cancellable cleans it up.** Abort in-flight
  requests, clear timers, remove listeners. An effect with no cleanup updates state after
  unmount.
- **Never render a success branch that a failed request can reach.** Do not infer success
  from "not loading and no error" — model the states so the impossible combination cannot
  be represented.
- Derive what can be derived. Do not mirror a prop or context value into state and sync it
  with an effect.

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

- Use the project's configured test runner and component-testing library. Follow the
  established test layout rather than introducing another one.
- Query by role and accessible name, not by test id or class. A test that cannot find an
  element by its accessible name has found a real accessibility bug.
- Cover the loading, error, and empty states, not only the populated one. Those are the
  branches users hit when something goes wrong.
- Cover form validation: a submit that should fail, and the error text a user would see.
- A project adopting this baseline may have no test harness at all. Setting one up is the
  first task, not a reason to skip the rule.

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

This stack ships code to browsers and deploys to hosting. Ask before:

- Any deploy or bucket sync, in any environment.
- Adding, upgrading, or removing a dependency.
- Changing the build, the container image, or the web server config.

Stack-specific traps worth knowing before you touch them:

- **Build-time environment variables are baked into the bundle and shipped to the
  browser.** Never put a secret in one. Declare their interface so a missing variable is a
  type error rather than a silent `undefined` that surfaces as a runtime failure.
- Read `tsconfig.json` before choosing syntax. Compiler flags are project configuration;
  never infer them from another Vite application or restate their current values here.

- Never read `.env` files, credential stores, key material, or CI secrets unless the task cannot be done otherwise and the user has asked for it.
- Never print a secret's value — not in output, not in logs, not in a comment, not in an error message. Refer to it by name.
- Never paste a secret, token, internal hostname, or customer data into an external service, a commit, an issue, or a PR description.
- Never write a credential into source. Read it from the environment at runtime, and add a placeholder to the example env file instead.
- If you encounter a committed secret, stop and report it. Do not quote the value in the report.

## Reporting

Report outcomes faithfully. If tests fail, say so and show the output. If you skipped a
step, say which. If something is done and verified, say so plainly without hedging. Never
describe work as complete when part of it is unfinished — say what is left and why.
