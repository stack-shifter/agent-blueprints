# Repository Guidelines

## Project Structure & Module Organization

This is a manual-copy vault for agent instructions, with no runtime application.

- `stacks/<stack>/AGENT.md` contains standalone project baselines. Sibling readmes explain stack selection.
- `sub-agents/claude/*.md` and `sub-agents/codex/*.toml` hold the same roles in each platform's own format; a role's two files must say the same thing.
- `snippets/<topic>/` stores canonical, atomic rule blocks grouped by concerns such as `git`, `testing`, and `safety`.
- `templates/` defines the required shape for each artifact type.

Update the root `README.md` index whenever adding a stack, persona, or snippet.

## Development and Validation Commands

There is no package manager, build step, or automated suite. Validate Markdown with:

```bash
git diff --check
rg "AUTHORING NOTES|<Stack Name>|role-name|Rule Title" stacks sub-agents snippets
git diff -- README.md stacks/ sub-agents/ snippets/ templates/
```

The first command catches whitespace errors. The scan finds template notes or placeholders
that escaped into an artifact. Review the diff to confirm indexes and canonical copies
agree.

## Writing Style & Naming Conventions

Use concise Markdown, imperative rules, and short sections. Wrap prose near the existing 80-character style when practical. Use kebab-case filenames such as `validation-review.md` and stack directories such as `python-fastapi/`.

Start snippets at `##`, keep one idea per file, and do not add a preamble. Claude sub-agents
follow the frontmatter and five-section order in `templates/sub-agent.template.md`; Codex
sub-agents follow `templates/codex-agent.template.toml` and use the same five-section body.
Names match filenames in each platform's required case, and Claude's `platform` matches its
directory. Delete all authoring-note comments when instantiating templates.

Most importantly, copied artifacts must be self-contained: never link to sibling repository files or depend on another fragment being pasted with them. Paste canonical snippet wording into applicable stack baselines rather than paraphrasing it.

## Testing Guidelines

Test by copying the changed artifact into an empty directory and reading it without repository context. Confirm instructions are complete, paths are destination-safe, frontmatter is valid, and shared wording matches its source snippet. A role's Claude and Codex files must state the same rules; any difference between them must be a format necessity, never a behavior change.

## Commits & Pull Requests

History is currently minimal, so follow `snippets/git/commit-message-format.md`: use `type(scope): summary`, for example `docs(rust): complete testing guidance`. Keep summaries imperative, lower case, under roughly 70 characters, and limit each commit to one logical change.

Pull requests should explain the intended copy target, list affected baselines, and note any deliberately duplicated wording. Link relevant issues and include before/after excerpts when a rule's behavior changes; screenshots are unnecessary for Markdown-only edits.
