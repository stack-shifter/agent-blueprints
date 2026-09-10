# Product Docs & Prototypes — Agent Baseline

Rules for working in a product documentation and prototyping repository. They apply to
every file unless a more specific instruction in the task overrides them.

This repository holds documents and static prototypes, not an application. Completed
prototype work never implies shipped software, and no runtime code belongs here unless
explicitly requested.

## Repository shape

A canonical product brief, a design and phase plan, a set of numbered specifications, and
a directory of self-contained HTML prototypes. Supporting material — a database profile, an
outreach document, reference assets — hangs off those.

Before working, read the entry-point README, the current plan, and the artifacts the task
touches. Do not assume a filename exists, and do not assume every document is
authoritative.

## Authority order

When two documents disagree, this is the order:

1. Explicit user-approved decisions
2. The canonical product brief
3. Reviewed prototypes
4. Product specifications
5. Supporting database, API, and architecture documents
6. The README, which is an entry point only

**Reviewed prototypes outrank written specifications.** If a prototype and a spec disagree,
the spec is the thing to fix. Never settle a conflict using the README, and surface a
conflict rather than resolving it silently.

Preserve user decisions and unrelated work. A legacy system is factual evidence of what
exists, not an automatic requirement.

## Planning

Before broad changes, update the repository's canonical plan — **whichever file already
holds it.** Never create a second plan file because guidance names one that does not exist;
two competing plans is the worst outcome available.

Keep the plan a current, phased checklist ordered by dependency. Record completion criteria
per phase, and remove superseded items rather than accumulating them. Work in small
reviewed slices.

## Prototypes

- Self-contained static HTML: embedded CSS and lightweight JavaScript, no build step, no
  external dependency, no CDN. It must run from a plain static server exactly as it sits.
- Fictional data only. Label data, credentials, tokens, and scenarios as fictional.
- Every page carries `noindex`, `nofollow`, and `noarchive`, plus accurate title,
  description, and viewport metadata. These pages get deployed to real URLs.
- Each page has one primary job.
- **Use explicit labelled controls. Never depend on hover, or on an unlabelled row click,
  for a critical action.**
- Model the states that matter: loading, empty, error, disabled, conflict, success. Keep
  filters and sorts functional, and preserve refresh-worthy state in the URL.
- Keep prototype-only scenario and debug controls out of production-facing views.

## The front door

Once there is more than one destination, maintain an index page as a review launcher —
**not an exhaustive sitemap.**

- One always-visible tile per primary user journey, opening a realistic default in a new
  tab with `rel="noopener"`. List only destinations that actually work.
- Gate reviewer-only states — administration, setup, auth and invite flows, error pages —
  behind a parameter such as `?admin=true`, **applied synchronously in `<head>`** so the
  gated content never flashes before it is hidden.
- Group several deterministic states for one destination in an accessible dropdown of
  shareable query-param links, rather than duplicating tiles or pages.
- **Never imply that a URL parameter grants authorization.** It is a display toggle on a
  publicly reachable page, and everything it hides is still in the DOM.
- Update the index whenever destinations or review states change, and verify the links.

## Drift control

Repeated application shell and shared-component code is **one canonical contract**, even
though self-contained prototypes force it to be duplicated across files.

- Maintain a clearly marked canonical block for duplicated shared CSS, and verify that its
  **normalized** contents match across every page that carries it. Files may be minified in
  one place and pretty-printed in another, so naive line-by-line diffing silently fails.
- Before completing a change, identify every page containing the affected pattern and
  compare final CSS, markup structure, responsive breakpoints, accessibility attributes,
  and interaction behaviour across all of them.
- **A visual comparison does not prove consistency.** Equal geometry conceals differences
  in shadows, borders, colours, transforms, focus behaviour, and responsive states. Check
  the source, and check computed styles at the widths that matter.
- Exercise shared interactions — drawers, dialogs, filters, action menus — on at least one
  page from each family.
- When you find drift, remediate the complete pattern across every affected screen rather
  than patching the page that reported it. Then re-run the comparison and look for
  obsolete rules that can still override the canonical one.
- Record an intentional exception in the plan before accepting it.

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

Verify at a wide desktop width, at the exact navigation breakpoint, and at a narrow mobile
width. Turn dense tables into cards or an intentional scroll region rather than letting the
page overflow horizontally.

## Specifications

- Write a spec only for behaviour that has been reviewed or explicitly approved.
- Keep one concise, feature-organized set with a single overview document as its table of
  contents. Number specs in delivery order.
- Describe complete workflows — browser behaviour, API responsibility, persistence — and
  state a shared rule once rather than repeating it per feature.
- **Document behaviour, invariants, boundaries, and non-obvious decisions. Do not
  transcribe markup.**
- Link to central sources rather than duplicating them. The specification folder is
  deliberately **not** a standalone portable bundle.
- **Record unresolved decisions explicitly before implementing the affected behaviour.**
  An open-questions list is load-bearing, not cruft — resolve entries one feature at a
  time, and never silently invent behaviour to clear one.

## Database profile

When a database is explicitly selected, that decision is closed. Do not add an alternative,
a selector, or a comparison document unless the user reopens it.

Begin the profile with machine-readable metadata naming the document type, the selected
profile and its status, the provider and access libraries, the runtimes, and the date it
was last reviewed.

Document tables and columns, keys and constraints, index coverage per access pattern,
transactions and locking, idempotency, outbox and processed-event handling, migrations,
retention, and connection budgets. Keep HTTP and domain types independent of database rows.

Treat it as a schema contract, not an applied migration, and reconcile it with the specs
whenever approved changes touch entities, ownership, lifecycle, queries, concurrency,
events, or file handling.

## Naming and numbering

- Lowercase hyphenated filenames. Two-digit ordering, with `00-` reserved for foundation
  states.
- Name a screen `NN-description.html` and a close companion `NNa-description.html`.
  **A letter suffix is not a substitute for choosing a real position** — if something needs
  its own number, give it one.
- Align spec and prototype numbers where practical, and link each feature spec to its
  primary prototype.
- Use one term per role, status, entity, and action. Prefer user-facing language over
  implementation names.
- **Do not casually renumber.** When you must, update every link, reading order, and route
  map in the same change.

## Outreach documents

Ground every claim in the canonical brief and the prototype that was actually built. Reuse
the product's real name, target customer, and capabilities.

**Never carry over claims, integrations, or positioning from a template or another
project's leftover copy** unless the current brief supports them. Preserve the document's
existing structure and channel variants rather than redesigning it, and use bracketed
placeholders for anything outside the repository's knowledge instead of inventing it.

## Prefer boring code

When several implementations work, choose the one a mid-level developer understands on the first read. Cleverness is a cost paid by every future reader.

- Prefer plain control flow — `if`, `for`, early returns — over deeply chained pipelines or condensed one-liners. Favor code that can be stepped through in a debugger.
- Follow the patterns already in the repository. Do not introduce a second architectural style alongside a working one.
- Some duplication is acceptable when it keeps logic readable and local. Do not abstract on the second occurrence; wait for the third.
- Build only what the current task requires. Never generalize for a hypothetical future caller.
- Do not introduce factories, builders, dependency-injection layers, or similar patterns unless the problem clearly demands one.
- Do not add a dependency for something the standard library already does.
- Added complexity is fine when it buys correctness, performance, or maintainability — but comment why it was worth the cost.

## Comments

Comment why, never what. If a comment restates the code, delete the comment or fix the name.

- Write a comment when the reason for the code is not recoverable from the code: a workaround, a non-obvious constraint, an ordering that matters, a deliberate deviation.
- Never leave commented-out code. Version control already remembers it.
- No changelog comments, no attribution comments, no `// TODO` without a concrete next action.
- Do not decide comment style per file. Match the closest existing peer in the same layer — a heavily annotated codebase and a bare one both have a house style.
- Keep coverage uniform within a peer group. Documenting some exported functions in a module while leaving equivalent ones bare is worse than documenting none.
- Doc comments on exported symbols state contract, not implementation: what it takes, what it returns, what it throws, what it assumes.

## Completion checks

Run the project's checks before reporting a change complete. Never describe work as done on checks you did not run.

- The default gauntlet is typecheck, lint, test, build. Run all of it after any change to runtime code.
- Scale the checks to the work. A documentation-only or comment-only change does not need the full suite — say which checks you skipped and why.
- Run the checks the repository actually defines. If a script is missing, say so rather than substituting an equivalent command.
- Fix what the checks report before moving on. Never leave a failing check for the reviewer to find.
- If a check cannot run in this environment, name it and say so plainly. Do not imply it passed.
- Report the outcome, not the intent: name the commands you ran and what they returned.

For this repository, checks are proportional to the work. Confirm that links, navigation,
URL state, and visible controls work; that specs match approved prototypes and the database
profile matches accepted contracts; that the README, status, ordering, and naming stay
accurate; that routes, roles, states, and terminology agree across documents; that no
deferred feature slipped into scope; and that only intended files changed.

Use targeted search, markdown-link inspection, local browser validation, and a whitespace
check as appropriate.

**Delete transient browser-automation artifacts before finishing** — session data,
snapshots, traces, temporary screenshots. Keep one only when it was explicitly requested as
a deliverable.

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

This repository publishes to live, publicly reachable URLs. Ask before:

- Running any deploy or sync script. They typically push to real hosting with `--delete`
  and invalidate a CDN cache.
- Renumbering or renaming artifacts that other documents link to.
- Rewriting a canonical source, or expanding one feature into a suite.
- Introducing runtime code or deployment resources that the task did not ask for.

Never leave a link to a file that does not exist, and never create a second canonical
document where one already exists.

- Never read `.env` files, credential stores, key material, or CI secrets unless the task cannot be done otherwise and the user has asked for it.
- Never print a secret's value — not in output, not in logs, not in a comment, not in an error message. Refer to it by name.
- Never paste a secret, token, internal hostname, or customer data into an external service, a commit, an issue, or a PR description.
- Never write a credential into source. Read it from the environment at runtime, and add a placeholder to the example env file instead.
- If you encounter a committed secret, stop and report it. Do not quote the value in the report.

Prototypes are deployed publicly: never put a real credential, customer name, or internal
hostname in one, even as placeholder content.

## Reporting

Report outcomes faithfully. If a check fails, say so and show the output. If you skipped a
step, say which. If something is done and verified, say so plainly without hedging. Never
describe work as complete when part of it is unfinished — say what is left and why.
