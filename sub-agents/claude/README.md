# claude/

Markdown personas that are **real Claude Code sub-agent configuration**. Copy one into a
destination project's `.claude/agents/` directory and it works as-is.

## The pairing rule

Every role lives twice: markdown here, TOML in [`../codex/`](../codex/). The two files
exist because the platforms read different formats, not because the roles differ — so
**they must say the same thing.** Change one and change the other in the same commit.

If a role genuinely only makes sense on one platform, it exists only there, and its
description should make the reason obvious.

## What belongs in the frontmatter

- `name` — kebab-case, matching the filename.
- `description` — third person, states when to invoke. This is the routing signal.
- `tools` — least privilege. Grant `Write`/`Edit` only to a persona that must mutate files;
  the Codex twin's `sandbox_mode` should agree with that choice.
- `model` — `inherit` unless there is a reason.
- `platform: claude` — must match this directory.

Start from [`../../templates/sub-agent.template.md`](../../templates/sub-agent.template.md).

## Why there is no shared directory

There used to be a `shared/` holding personas said to work on both platforms. That premise
did not survive contact with the formats: Codex loads TOML, so a markdown file cannot be
dropped into `.codex/agents/` and work. A directory whose name promised both platforms
delivered one.

Two directories, each named for the platform it actually loads on, keeps the vault's one
rule intact — copy a file where its directory says, and it works.
