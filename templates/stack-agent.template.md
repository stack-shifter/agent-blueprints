# <Stack Name> — Agent Baseline

Rules for working in a <stack> project. They apply to every file unless a more specific
instruction in the task overrides them.

**Assumed majors:** <Framework N, Language N>. Only a few rules below depend on a major,
and each says so where it does. If the project is on a different major, the project wins —
change this file rather than follow a rule that no longer applies.

## Project shape
What to assume about layout, entry points, and where things live. Two or three sentences,
not a directory listing — the agent can read the tree.

## Toolchain
Name the package manager and the checks that matter. Include an exact command only when it
is invariant across the stack; otherwise tell the agent which manifest or workspace
configuration defines it. Say what to do when a script is missing: report it, never invent
an equivalent.

<!-- THE STACK-SPECIFIC SECTIONS GO HERE — this is the bulk of the file and the
     reason it exists. Whatever this stack gets wrong that others do not: its
     construct API, its layering, its routing, its data access, its drift
     control. Give each its own H2. -->

## Prefer boring code
## Strict typing
## Vertical code layout
## Naming
## Comments
## Accessibility
## Testing
## Verification
## Version control
## Boundaries

## Reporting
Report outcomes faithfully. Say what failed, what was skipped, and what is left.

<!-- ─────────────────────────────────────────────────────────────────────────
AUTHORING NOTES — delete this block when you fill the template in.

THIS FILE GETS COPIED OUT WHOLE.
  It is the entire instruction set for a project. It must stand alone with
  nothing else beside it:
    - no "see snippets/..." references — paste the text instead
    - no assumption that another file was pasted above or below it
    - no base layer; there isn't one
    - no mention of the repository it was derived from

THE SECTION LIST ABOVE IS A SPINE, NOT A SCHEMA.
  Unlike the sub-agent template, this one does not prescribe a fixed set. Real
  baselines run 13-19 sections. The listed ones are what most carry, in the
  order they carry them; the stack-specific block in the middle is the point.
  Drop what does not apply — a docs stack has no Toolchain and no Testing.

BUILD IT BY PASTING FROM snippets/, PROGRAMMATICALLY.
  The lower sections are snippet bodies with their "## " heading stripped.
  Assemble the file with a script that reads snippets/ rather than copying by
  hand, so byte-fidelity holds by construction instead of by carefulness.

THE SNIPPET SUBSET MAY BE PARTIAL.
  Paste only the snippets that apply. A documentation stack has no typed source
  and no test suite, so it carries neither typing/ nor test-discipline. When you
  omit one, say so and why in the sibling README, or the next reader cannot tell
  deliberate from forgotten.

ONE PLACE FOR VERSIONS.
  Put the assumed majors in the line near the top and nowhere else. Do not
  scatter minimum versions through the body — a package manager already enforces
  those, and they rot. State a version inline only where the correct code
  genuinely differs by major, and then say which major and what to check.

PROJECT CONFIGURATION IS AUTHORITATIVE.
  Do not copy formatter, linter, compiler-flag, coverage-threshold, or script-
  behavior values into a baseline merely because a reference project uses them.
  Tell the agent to read the relevant project configuration. Keep a value only
  when it is a deliberate stack invariant, and document it in the sibling README.

WHAT BELONGS HERE
  Only rules true for EVERY project on this stack. Anything specific to one
  project belongs in that project's copy, edited after copying — not upstream.

WHAT DOES NOT BELONG HERE
  Guidance about which baseline to choose, and any note about how this baseline
  disagrees with the codebase it came from. Both go in the sibling README.md,
  which is vault-internal and never copied out.

SIBLINGS, NOT LAYERS
  If this stack is a variation of another, write it as a full standalone
  baseline that duplicates the shared text. Duplication is the accepted cost of
  "copy one file, done". Generate the pair from shared blocks so the superset
  relationship holds by construction.

  But if two baselines are so close that choosing between them is unclear, they
  should be one baseline.
────────────────────────────────────────────────────────────────────────── -->
