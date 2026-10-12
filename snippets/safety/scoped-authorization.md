## Scoped authorization

Explicit authorization covers the named scope and targets. Do not ask again for
ordinary edits or operations already approved. Read existing files first and
preserve unrelated work. Approval to implement phases does not authorize deployment
or writes against live data.

- Obtain explicit authorization for deployment/destruction, live-data writes,
  migration execution, dependency changes, history rewrites, force pushes,
  consequential deletion, external messages/PRs, or security-sensitive operations.
- Requests covering schema, infrastructure, CI, build, hosting, or migration
  authoring authorize those edits, not execution against a live target.
- Ask when target, risk, or scope materially expands. Ambiguous permission is not
  approval. Follow higher-priority per-action confirmation requirements.
