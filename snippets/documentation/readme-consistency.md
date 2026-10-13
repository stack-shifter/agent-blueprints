## README Consistency Standard

Give readers a concise, familiar entry point. Standardize organization, not content,
tooling, or architecture; preserve useful repository-specific information.

### Recommended section order

Use these names and this order where applicable; omit irrelevant sections.

1. **Purpose** — Briefly explain what the repository does, who uses it, and its
   responsibility within the broader application.
2. **Architecture / Repository Overview** — Describe major components and connections;
   for docs repositories, describe organization and sources of truth. Include a small
   diagram only when it clarifies a relationship; accuracy matters more than format.
3. **Prerequisites and Configuration** — Identify tools, external resources, and required
   values before commands that need them. Separate local and deployment requirements;
   explain value sources, generated values, and public exposure. Never include secrets.
4. **Local Setup and Usage** — Give complete, ordered steps using actual package scripts,
   including tests/checks. For docs repositories, explain navigation, editing, preview,
   and validation instead of inventing application workflows.
5. **Deployment / Publishing** — Where applicable, separate Development and Production,
   first-time preparation and routine releases. Explain triggers, configuration,
   migration ordering, and publishing risks; link detailed runbooks.
6. **Teardown / Cleanup** — Show supported commands and exact targets where applicable.
   State what is deleted, retained, imported, or needs separate cleanup. Never imply
   stack destruction removes every related resource.
7. **Operational Notes and Limitations** — Include troubleshooting, compatibility, and
   deferred capabilities. Distinguish implemented behavior, known limitations, and
   checks or deployment behavior not yet verified.
8. **References** — Link useful specs, testing guides, contracts, and runbooks. Reuse
   existing indexes rather than adding exhaustive document catalogs.

### Content and editing rules

- Review the existing README and current source before reorganizing. Preserve important
  information; remove only duplication, obsolete guidance, or inaccuracies. When moving
  detail to a runbook, retain a short explanation and a working README link.
- Preserve API endpoints, authentication, configuration, database deployment,
  workers/retries, test prerequisites, and migration safety when applicable.
- Preserve UI routes, authentication/loading, environment variables, build labels,
  API connection requirements, and deployment details when applicable.
- Preserve docs requirements, prototypes, company standards, ownership, navigation,
  editing, and validation conventions. Put route/endpoint references near the overview
  or in a clearly named section nearby; do not force awkward placement.
- Keep commands copyable, correctly scoped, and consistent with scripts and pipelines.
  Do not impose another repository's framework, runner, environments, or architecture.
- Avoid completed-cycle handoffs, repeated spec narratives, and speculative features
  presented as current behavior. README refresh work does not authorize runtime code,
  infrastructure, dependency, or deployment changes.

### Verification

Verify commands, routes, configuration, links, and operational claims against current
source. Run applicable Markdown formatting and whitespace checks. Report unverified
statements plainly; local tests alone do not establish production readiness.
