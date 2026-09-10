# templates

Vault-internal. Never copied out.

These four files define the required shape of every artifact in the vault. They are not
content — nothing here is useful to a project. You instantiate one to create a vault file,
and it is that vault file which later gets copied into a project.

```
templates/  →  stacks/ sub-agents/ snippets/  →  a project
   shape            the vault artifact            the copy
```

## Which template

| Template | Produces | Which lands in |
|---|---|---|
| [`stack-agent.template.md`](stack-agent.template.md) | `stacks/<stack>/AGENT.md` | `.claude/CLAUDE.md` |
| [`sub-agent.template.md`](sub-agent.template.md) | `sub-agents/claude/<role>.md` | `.claude/agents/` |
| [`codex-agent.template.toml`](codex-agent.template.toml) | `sub-agents/codex/<role>.toml` | `.codex/agents/` |
| [`snippet.template.md`](snippet.template.md) | `snippets/<topic>/<rule>.md` | an existing agent file |

Choose by paste size. One rule is a snippet. One job with a defined return shape is a
sub-agent. Everything an agent must know to work in a project is a stack baseline.

If you are unsure between a snippet and a stack section: write the snippet. A stack
baseline's lower half is snippet bodies anyway, and a rule that lives in `snippets/` can
be pasted into every baseline that wants it.

## How to use one

1. Copy the template to its destination path.
2. Fill it in.
3. **Delete the authoring-notes comment block.** It is the last thing in every template,
   fenced in a comment, and it must not survive into the artifact.
4. Add a row to the root [`README.md`](../README.md) index.

Then verify:

```bash
rg "AUTHORING NOTES|<Stack Name>|role-name|Rule Title" stacks sub-agents snippets
git diff --check
```

Both should come back empty. The first catches a template that was filled in but not
cleaned up; the second catches whitespace damage.

## Two are schemas, one is a spine

`sub-agent.template.md` and `codex-agent.template.toml` are **schemas**. Five sections,
fixed order, every time. That rigidity is the point — it is what lets you diff two
personas by eye and see only what differs.

`stack-agent.template.md` is a **spine**. Real baselines run 13–19 sections and no two
carry the same set. The listed sections are what most carry, in the order they carry
them; the stack-specific block in the middle is the reason the file exists. Drop what
does not apply.

`snippet.template.md` sits between the two: the shape is fixed (one `##` heading, no
preamble, bullets), the length is not.

## The pairing rule

A sub-agent is never one file. Every role exists twice — markdown in `sub-agents/claude/`,
TOML in `sub-agents/codex/` — because Codex does not read markdown and Claude does not
read TOML. The two must say the same thing. Write both, change both, in the same commit.

Keep the permission grants in agreement too: a persona that must mutate files carries
`Write`/`Edit` in its Claude `tools` and `sandbox_mode = "workspace-write"` in its Codex
twin. Auditors and reviewers carry neither.

## The rule behind all four

**Every artifact must be complete and correct after one copy.** Each template restates this
in its own terms, because it is the only invariant the vault has:

- No `see snippets/...` references — paste the text instead.
- No assumption that another file was pasted above, below, or beside it.
- No base layer to inherit from; there isn't one.
- No mention of the vault or the repository the rules were derived from.

Duplication across artifacts is the accepted cost. When two baselines share text, generate
both from the same blocks so they stay byte-identical by construction rather than by
carefulness.

## What does not belong in an artifact

Guidance about *which* baseline or persona to choose, and any note about how an artifact
disagrees with the codebase it came from. Both go in the sibling vault-internal `README.md`
— this file's counterparts in `stacks/<stack>/` and `sub-agents/<platform>/` — which is
never copied out.
