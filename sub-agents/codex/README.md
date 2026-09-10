# codex/

Agents for OpenAI Codex. **These are TOML, not markdown** — copy one into the destination
project's `.codex/agents/` directory and it loads as-is.

## The format

```toml
name = "role_name"                    # snake_case; the filename stays kebab-case
description = "What it does, plus when to invoke it."
model = "…"                           # environment-specific — see below
model_reasoning_effort = "medium"     # low | medium | high
sandbox_mode = "read-only"            # omit only for an agent that must write

developer_instructions = """
The whole prompt: mission, rules, process, output contract, boundaries.
"""
```

`name` must be the snake_case form of the filename — `validation-review.toml` declares
`name = "validation_review"`. A mismatch is how you spot a misfiled copy.

`model` is the one line that is environment-specific. Check it against what the destination
actually has available before relying on a copied file; everything else transfers unchanged.

Start from `../../templates/codex-agent.template.toml`.

## The pairing rule

Every role lives twice: TOML here, markdown in [`../claude/`](../claude/). The two files
exist because the platforms read different formats, not because the roles differ — so
**they must say the same thing.** Change one and change the other in the same commit.

`sandbox_mode` should agree with the Claude twin's `tools` grant: `read-only` for a persona
that audits or reviews, `workspace-write` for one that must mutate files.

If a role genuinely only makes sense on one platform, it exists only there, and its
description should make the reason obvious.

## Why there is no shared directory

There used to be a `shared/` holding personas said to work on both platforms. That premise
did not survive contact with the formats: Codex loads TOML, so a markdown file cannot be
dropped into `.codex/agents/` and work. A directory whose name promised both platforms
delivered one.

Two directories, each named for the platform it actually loads on, keeps the vault's one
rule intact — copy a file where its directory says, and it works.
