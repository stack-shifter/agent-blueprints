# claude/

Personas that depend on Claude or Claude Code capabilities and would not work pasted
elsewhere.

## What belongs here

A file earns a place here only if it relies on something Claude-specific:

- Spawning and coordinating sub-agents
- Tool grants in frontmatter (`tools:`), or a specific `model:`
- Skills, hooks, or MCP server wiring
- A strategy built around a large context window — reading broadly before narrowing

If a persona would work unchanged after stripping the frontmatter, it belongs in
`../shared/` instead.

## The override rule

A role name appears here only if it is **absent from `../shared/`**, or is a **deliberate
override**. An override must open with a line stating what it changes relative to the
shared version and why the shared version was insufficient.

Without that, the two copies diverge silently and you stop trusting either.

## Frontmatter

The frontmatter here is real Claude Code sub-agent configuration. Copy these files into a
destination project's `.claude/agents/` directory and they work as-is.
