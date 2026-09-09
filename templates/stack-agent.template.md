# <Stack Name> — Agent Baseline

Rules for working in a <stack> project. Applies to every file in this repository
unless a more specific instruction overrides it.

## Project shape
What the agent should assume about layout, entry points, and where things live.
Two or three sentences, not a directory listing — the agent can read the tree.

## Toolchain
The exact commands for build, test, lint, and typecheck. Name the package manager
and pin the expectation (e.g. "use pnpm, never npm"). Wrong-tool invocations are
the most common and most annoying failure, so be explicit.

## Code style
Formatting, naming, and structural conventions. Paste from snippets/style/ and
snippets/typing/ rather than paraphrasing, so the wording matches other baselines.

## Testing
What must be tested, what framework, and what the agent should do when it cannot
run the tests.

## Version control
Commit granularity, message format, branch discipline. Paste from snippets/git/.

## Boundaries
What requires asking first: destructive commands, dependency additions, schema
changes, anything touching credentials or CI. Paste from snippets/safety/.

<!-- ─────────────────────────────────────────────────────────────────────────
AUTHORING NOTES — delete this block when you fill the template in.

THIS FILE GETS COPIED OUT WHOLE.
  It is the entire instruction set for a project. It must stand alone with
  nothing else beside it:
    - no "see snippets/..." references — paste the text instead
    - no assumption that another file was pasted above or below it
    - no base layer; there isn't one

WHAT BELONGS HERE
  Only rules true for EVERY project on this stack. Anything specific to one
  project belongs in that project's copy, edited after copying — not upstream.

WHAT DOES NOT BELONG HERE
  Guidance about which baseline to choose. That goes in the sibling README.md,
  which is vault-internal and never copied out.

BUILDING IT
  Assemble by pasting from snippets/. The snippet is the canonical wording of a
  rule; pasting rather than paraphrasing is what keeps the same rule reading
  identically across every baseline that carries it.

SIBLINGS, NOT LAYERS
  If this stack is a variation of another (nextjs vs node), write it as a full
  standalone baseline that duplicates the shared text. Duplication is the
  accepted cost of "copy one file, done".

  But if two baselines are so close that choosing between them is unclear, they
  should be one baseline.
────────────────────────────────────────────────────────────────────────── -->
