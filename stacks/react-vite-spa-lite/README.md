# react-vite-spa-lite

Vault-internal. Never copied out.

## Pick this when

You are building a React single-page application and bringing your own choices for auth,
styling, forms, and data fetching — or joining a project that already made them.

## Assumes

- React, Vite, TypeScript strict, npm, and React Router. Installed configuration
  and types determine supported versions and router interfaces.
- Preserve existing feature- or purpose-based organization, routing, and providers.
- The repository chooses authentication, styling, forms, fetching, and its test tools.
- Automated tests use isolated configuration and controlled external boundaries.
- WCAG conformance applies only when the project specifies a version and level.

## Pick the sibling instead when

You want the house set decided for you — Cognito hosted-UI auth, Bootstrap, React Hook
Form with Zod. That is [`react-vite-spa-full`](../react-vite-spa-full/).

The two are **peers, not layers**. `full` is a standalone superset that repeats this
file's text; it is not an overlay to paste on top of this one.

## Snippet subset

Carries seventeen snippets, including test isolation, deferred work, and spec/plan
scope. Uses `accessibility/project-requirements` and `safety/scoped-authorization`
in place of unconditional WCAG conformance and repeated approval for ordinary edits.
Omits Python typing because this is a TypeScript stack. Shared wording is pasted
in full; the baseline has no dependency on these vault paths.

All stacks carry the canonical README consistency standard in full.

## Status

**Complete.**
