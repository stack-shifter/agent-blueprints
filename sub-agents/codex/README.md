# codex/

Personas shaped for OpenAI Codex and similar CLI setups, where the prompt structure or the
assumed capabilities differ enough that a shared file would mislead.

## What belongs here

- Instructions assuming a patch or diff-oriented workflow rather than file-editing tools
- Prompts written for a smaller effective context, front-loading constraints and repeating
  the critical ones
- Anything assuming no sub-agent spawning and no tool grants

If a persona would work unchanged on both platforms, it belongs in `../shared/` instead.

## The override rule

A role name appears here only if it is **absent from `../shared/`**, or is a **deliberate
override**. An override must open with a line stating what it changes relative to the
shared version and why the shared version was insufficient.

## Frontmatter

The frontmatter in these files is vault metadata only — it exists so the file is indexable
and so a misfiled copy is obvious. Strip it when pasting into Codex. `tools:` and `model:`
are omitted here because they mean nothing on this platform.
