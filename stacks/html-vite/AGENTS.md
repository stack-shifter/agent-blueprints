# Plain HTML (Vite) — Agent Baseline

Rules for working in a plain HTML project built with Vite, CSS, and vanilla
JavaScript. They apply to every file unless a more specific instruction in the
task overrides them.

Use HTML for document structure, CSS for presentation, and JavaScript modules for
behavior. Keep the stack framework-free unless the task explicitly changes that
choice. Styling libraries, authentication, and test tools are project decisions.

Read manifests, lockfiles, and installed configuration for supported versions and
interfaces. Never infer them from another project.

## Project shape

Read the tree and Vite configuration before assuming a layout. HTML files are
entry points; preserve the configured project root and existing single-page or
multi-page structure. Keep authored HTML, styles, scripts, and assets separate
from generated build output. Never edit generated output by hand.

## Toolchain

Use the package manager selected by the repository's manifest and lockfile.
Read `package.json` for install, development, build, preview, lint, format, and test
commands. Script names and composition are project configuration. If a script is
missing, report it rather than inventing an equivalent command.

Read the Vite and formatter configuration before changing it. Keep existing
plugins and build settings unless the task requires a change. Do not introduce a
framework, TypeScript migration, CSS processor, or test runner as incidental work.
Run every configured check that applies; never infer that one command includes
another. Typechecking applies only when the project configures it.

## HTML and navigation

- Author meaningful content and document structure in HTML. Use JavaScript only
  for behavior that requires it; do not rebuild a static page from a large string.
- Preserve the document language, viewport metadata, descriptive page title,
  landmarks, heading hierarchy, and unique element IDs.
- Use anchors with real destinations for navigation and buttons for actions.
  Give non-submit buttons an explicit `type="button"` inside forms.
- Prefer native elements and browser behavior before building custom controls.
  Keep essential content and navigation usable without JavaScript where practical.
- Preserve the existing URL model. Do not add a client router or intercept links
  just to turn a multi-page site into a single-page application.
- When adding an HTML page, confirm it is included by the configured production
  build and reachable on the intended host. A working development URL is not proof.

## CSS and responsive layout

- Follow the existing stylesheet organization, class naming, and design tokens.
  Reuse custom properties for shared values; keep one source for each token.
- Prefer normal flow, Flexbox, and Grid. Avoid absolute positioning for primary
  layout and fixed dimensions that clip content or prevent text from wrapping.
- Check narrow and wide layouts, long content, zoom, and visible focus. Preserve
  usable controls and readable content without unnecessary horizontal scrolling.
- Keep selectors local and understandable. Do not solve specificity problems with
  escalating selector chains or unexplained `!important` declarations.
- Respect reduced-motion preferences when adding animation. Use JavaScript-driven
  layout or animation only when CSS cannot express the required behavior cleanly.

## JavaScript and DOM behavior

- Use native ES modules and load page entry scripts with `type="module"`. Keep
  logic in focused modules; avoid inline event attributes and mutable globals.
- Query within the relevant page or feature root. Handle optional elements and
  fail clearly when a required element is missing before reading its properties.
- Register handlers with `addEventListener`. Make initialization safe against
  duplicate listeners; remove listeners, timers, and pending work when a feature
  is torn down or replaced.
- Keep state minimal and local. Derive display values instead of storing copies
  that can drift. Update the affected DOM without replacing focused controls.
- Use `textContent`, DOM creation methods, or cloned templates for dynamic content.
  Never interpolate untrusted content into `innerHTML` or HTML attributes.
- Prefer native form constraints. Handle the form's submit event so keyboard
  submission works; preserve entered values and associate errors with fields.
- For asynchronous actions, show pending, success, and failure explicitly. Prevent
  duplicate submissions, check unsuccessful HTTP responses, and handle rejected
  requests. Cancel or ignore stale results before they overwrite newer state.
- Browser checks control presentation; enforce authorization and trusted input
  validation on the server when a backend is present.

## Vite assets and configuration

- Use Vite-managed imports and HTML/CSS asset references for files that belong in
  the build graph. Use the configured public directory for files that must keep
  their names or be copied without transformation.
- Reference public assets by their served URL, without the public directory name.
  Respect the configured deployment base when constructing URLs in JavaScript.
  Verify nested pages and subpath hosting against the built output.
- Keep application scripts in the module graph. Do not bypass the build with
  unmanaged script tags or runtime CDN dependencies as an incidental shortcut.
- Treat client-exposed configuration as public. `VITE_` variables are exposed by
  default; inspect configured prefixes before adding values. Never include
  credentials in browser code, HTML, public assets, or client configuration.
- Validate required configuration and parse string values explicitly. Build-time
  values require a rebuild to change; do not describe them as runtime secrets.
- Use the configured preview workflow to inspect production output when relevant.
  A local preview is verification, not a production hosting service.

## Accessibility

Follow explicit project accessibility requirements. Apply WCAG conformance only
when specified; use the requested version and level. Formal certification and
comprehensive audits require an explicit scope.

- Use semantic HTML and keyboard-operable controls with visible focus.
- Give controls accessible names and images appropriate alt text.
- Associate form fields with labels and validation messages.
- Do not rely on colour alone to communicate meaning.
- Implement additional accessibility behavior required by the feature specification.

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

Use the project's existing test tools and layout. Add or update meaningful tests
for changed JavaScript behavior, especially form validation, asynchronous error
states, stale requests, and repeated initialization. Assert observable DOM output
and interactions; prefer roles and accessible names when the tools support them.

For static HTML or CSS changes, verify the affected pages in a browser: check
navigation, keyboard use, focus, responsive layout, and asset loading. Do not add
an automated suite solely to mirror static markup. If no test harness exists,
report that gap and the manual checks performed; agree on setup scope before
adding dependencies. Never weaken existing assertions to make a suite pass.

## Mock boundaries

Mock at external boundaries and deliberate architectural seams, nowhere else.

- Mock what you do not own: network calls, clocks, randomness, the filesystem, and third-party services. Mock an internal collaborator only through an intentional seam such as an injected repository or composition root.
- Never mock the thing under test or an internal implementation detail merely to make a test easier to write. That couples the test to the implementation and it will pass while the code is broken.
- Prefer a real in-memory implementation (an actual sqlite, a fake repository) over a mock with scripted return values.
- If a test needs more than a couple of mocks to run, the design is telling you the unit has too many dependencies. Fix the design instead of adding mocks.
- Never assert that a mock was called as the primary assertion. Assert on the resulting state or output.

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

For this stack, run the configured production build and inspect affected pages
through the available browser workflow. Check for browser errors, missing assets,
broken links, and failed interactions. Report unavailable browser verification
plainly; a successful build alone does not verify layout or behavior.

## Version control

- Never commit directly to the default branch. Branch first, always.
- Branch names: `type/short-description` in kebab-case — `fix/token-refresh-race`.
- Commit and push only when explicitly asked. Finishing a change is not permission to commit it.
- Never amend, rebase, force-push, or reset a branch that has been pushed, unless explicitly told to.
- Never use `git checkout .`, `git reset --hard`, or `git clean` to discard work you did not create. Uncommitted changes in the tree may be the user's.
- Never commit generated artifacts, dependency directories, or anything matched by the ignore file. Never commit a credential, even to a private repo.

## Commit messages

Format: `type(scope): summary` — `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`.

- Summary in the imperative mood, lower case, no trailing period, under ~70 characters. "add retry to fetch client", not "Added retries."
- The summary says what changed. The body, when present, says why — the problem, the constraint, the alternative rejected. Never restate the diff in prose.
- One logical change per commit. If the summary needs "and", split the commit.
- Never mix a refactor with a behavior change in one commit. They need different review attention.
- Reference the issue in the body, not the summary.

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

## Secrets

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

Report outcomes faithfully. Say what changed, which checks ran and their results,
what failed or was skipped, and what remains unverified. Distinguish build success,
browser verification, and deployment readiness.
