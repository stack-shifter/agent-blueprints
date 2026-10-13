# Repository Guidelines

## Project Structure & Module Organization

This is a manual-copy vault for agent instructions, with no runtime application.

- `stacks/<stack>/AGENTS.md` contains standalone project baselines. Sibling readmes explain stack selection.
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

## README Consistency Standard

### Goal

Give each repository a familiar, concise entry point without removing useful
information. Standardize organization, not identical content, tooling, or
architecture. Readers should find the same kinds of information in familiar places.

### Recommended section order

Use these section names and this order where applicable. Omit irrelevant sections
rather than inventing commands or workflows.

1. **Purpose** — Explain what the repository does, who uses it, and its
   responsibility within the broader application in a short paragraph.
2. **Architecture / Repository Overview** — Describe major components and their
   connections. Include a small diagram when it clarifies relationships. For
   documentation repositories, explain document organization and sources of truth.
3. **Prerequisites and Configuration** — Identify tools, external resources, and
   required configuration. Separate local-development and deployment requirements.
   Explain where values come from, which are generated, and which are publicly
   exposed. Never include real secrets.
4. **Local Setup and Usage** — Give complete, ordered steps using actual package
   scripts, including applicable tests and checks. For documentation repositories,
   explain navigation, editing, previewing, and document validation.
5. **Deployment / Publishing** — Where applicable, separate Development and
   Production and distinguish first-time preparation from routine releases.
   Explain triggers, configuration, migration ordering, and publishing risks.
   Link detailed runbooks rather than reproducing them.
6. **Teardown / Cleanup** — Where applicable, show supported cleanup commands and
   their targets. State what is deleted, retained, imported, or requires separate
   cleanup. Never imply a stack destroy removes every related resource.
7. **Operational Notes and Limitations** — Capture important behavior,
   troubleshooting, compatibility constraints, and deferred capabilities.
   Distinguish implemented behavior, known limitations, and checks or deployment
   behavior that have not been verified.
8. **References** — Link useful specifications, testing guides, contracts, and
   runbooks. Avoid exhaustive document catalogs when an existing index serves that
   purpose.

### Repository-specific content

Preserve useful details appropriate to the repository:

| Repository | Important information to preserve |
|---|---|
| API | Endpoint reference, authentication, configuration, database deployment, workers/retries, testing prerequisites, migration safety |
| UI | Route reference, authentication/loading behavior, environment variables, build labels, API connection requirements, deployment |
| Docs | Product requirements, prototypes, company standards, document ownership, navigation, editing and validation conventions |

Place route and endpoint references under the overview or a clearly named section
nearby. Do not force useful material into an awkward location for consistency.

### Editing rules

- Review the existing README and current source before reorganizing.
- Preserve important information. Remove only duplication, obsolete guidance, or
  inaccuracies.
- When moving detailed material to a runbook, retain a short explanation and a
  working link in the README.
- Keep commands copyable, correctly scoped, and consistent with actual package
  scripts and pipelines. Explain prerequisites before commands that require them.
- Avoid completed-cycle handoffs, speculative features presented as implemented,
  and repeated spec narratives.
- Do not impose another repository's framework, test runner, environment naming,
  or deployment architecture.
- Use diagrams only when they clarify an important relationship. Prefer accuracy
  over a particular format.
- README refresh work does not authorize changes to runtime code, infrastructure,
  dependencies, or deployment.

### Verification

Confirm commands, routes, configuration, links, and operational claims against
current source. Run applicable Markdown formatting and whitespace checks. Report
unverified statements plainly; never claim production readiness solely from local
tests. For this vault, validate navigation and document workflows rather than
inventing application setup or deployment instructions.

## Commits & Pull Requests

History is currently minimal, so follow `snippets/git/commit-message-format.md`: use `type(scope): summary`, for example `docs(rust): complete testing guidance`. Keep summaries imperative, lower case, under roughly 70 characters, and limit each commit to one logical change.

Pull requests should explain the intended copy target, list affected baselines, and note any deliberately duplicated wording. Link relevant issues and include before/after excerpts when a rule's behavior changes; screenshots are unnecessary for Markdown-only edits.
