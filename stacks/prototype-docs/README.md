# prototype-docs

Vault-internal. Never copied out.

## Pick this when

You are working in a product documentation and prototyping repository — a canonical brief,
a phased design plan, numbered specifications, and self-contained static HTML prototypes
with a gated review launcher. No application code.

## Assumes

- Documents and prototypes are the deliverable; nothing here builds or deploys an app.
- Prototypes are self-contained static HTML with embedded CSS and light JS — no build step,
  no external dependency, fictional data, `noindex` metadata.
- A single canonical plan file already exists under some name.
- If a database is documented, one profile is selected and that decision is closed.

## Pick a different stack when

You are writing application code. The specs produced here hand off to an implementation
repository, which gets its own baseline — a serverless API, a React SPA, an Angular SPA.
That separation is deliberate: completed prototype phases never imply shipped software.

## Note on the snippet subset

This is the only baseline that pastes **part** of `snippets/`, because most of them assume
code. It carries accessibility, prefer-boring-code, comments, verify-before-done, both git
snippets, and both safety snippets.

It deliberately omits:

- `typing/*` — there is no typed source here.
- `testing/test-discipline` and `testing/no-mock-overuse` — no test suite exists and none
  is expected. Completion checks take their place.
- `style/vertical-code-layout` and `style/naming-conventions` — identifier-level rules.
  This repository's naming discipline is file-level and gets its own section instead.

If a later docs repo grows typed source or a test suite, paste those snippets in then.

## Note on prescription

The baseline requires one canonical plan file **whatever it is named**, rather than naming
a specific one. The reference repository's own guidance names a plan file that does not
exist there — the actual plan lives elsewhere — so a literal reading produces two competing
plans. That correction is the highest-value rule in this file.

## Status

**Complete.** Derived from a production product-documentation repository.
