---
name: role-name
description: What this agent does, in the third person, plus when to invoke it. This is the routing signal — most of the file's value is here.
tools: Read, Grep, Glob, Bash
model: inherit
platform: shared
---

# Role Name

## Mission
One or two sentences. What this agent is for, and what "done" means.

## Operating rules
- Hard constraints, imperative voice, one per line.
- Include the negative rules — what it must NOT do is usually the load-bearing half.
- Prefer rules that are checkable over rules that are aspirational.

## Process
1. Ordered steps, and only when order actually matters.
2. Keep to five or fewer.
3. More than five means this is a workflow, not a persona — split it.

## Output contract
Exactly what this agent returns: format, ordering, severity labels, what it omits.
Specific enough that two runs on the same input are comparable.

## Boundaries
When to stop and hand back rather than proceeding. What is explicitly out of scope.

<!-- ─────────────────────────────────────────────────────────────────────────
AUTHORING NOTES — delete this block when you fill the template in.

FRONTMATTER
  name         kebab-case, matches the filename. A role, not a personality.
  description  Third person, states WHEN to invoke. This is what a dispatcher
               reads to choose this agent; write it for that reader.
  tools        Least privilege. Omit the key entirely to inherit everything.
               Grant Write/Edit only to agents that must mutate files.
  model        inherit | sonnet | opus | haiku
  platform     shared | claude | codex — must match the directory this file
               lives in. A mismatch is how you spot a misfiled copy.

  Codex has no use for this frontmatter; it is safe to strip when pasting there.
  That is what lets one shared/ file serve both platforms.

SECTIONS
  Five sections, in the order above, always. Fixed order is what lets you diff
  two personas by eye.

  No preamble, no role-play flourish. "You are a world-class..." earns nothing.

  Under ~60 lines. Longer means it wants to be a stack baseline or a set of
  snippets instead.

  Output contract is mandatory. An agent with no defined return shape produces
  results you have to re-read every time.

  Boundaries are mandatory for anything that can write files, run commands, or
  spend money.

PLACEMENT
  Default to sub-agents/shared/. Put a file in claude/ or codex/ only if it is
  absent from shared/, or is a deliberate override — and an override must open
  with a line saying what it changes and why shared/ was insufficient.

SELF-CONTAINMENT
  This file gets copied out of the vault on its own. It must make sense with
  nothing else beside it. Never reference a sibling path; paste the text.
────────────────────────────────────────────────────────────────────────── -->
