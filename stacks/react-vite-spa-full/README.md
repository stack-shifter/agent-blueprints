# react-vite-spa-full

Vault-internal. Never copied out.

## Pick this when

You are building a React single-page application on the house stack: Cognito hosted-UI
auth, Bootstrap with SCSS, React Hook Form with Zod. Typically an internal admin console
sitting behind Cognito.

## Assumes

Everything [`react-vite-spa-lite`](../react-vite-spa-lite/) assumes, plus:

- Amplify configured once at the entry point; token refresh delegated to the library.
- Bootstrap through SCSS, one co-located stylesheet per component, BEM-ish namespacing.
- React Hook Form with a Zod resolver; schemas in one validation directory.

This file repeats lite's text in full rather than layering on it — copy this one alone and
you have everything.

## Pick the sibling instead when

The project brings its own auth, styling, or forms.

## Note on prescription

This baseline deliberately disagrees with the production app it was derived from, in named
places. It requires effect cleanup for cancellable work, forbids reading state after
`await`, forbids state setters during render, and requires errors to be associated with
their fields — each correcting a real defect in the reference implementation.

It also **declines** two rules that app's own guidelines state, because the code is right
and the doc is wrong: inputs stay uncontrolled via `register()`, and co-located global
stylesheets with BEM namespacing are kept instead of CSS Modules.

Installed router interfaces determine imports, middleware, and lazy-loading APIs.
Preserve established architecture rather than migrating it to match an example.
WCAG conformance applies only when the project specifies a version and level.

## Snippet subset

Carries the same sixteen snippets as lite, with shared wording pasted in full.
Uses conditional accessibility and scoped authorization, plus test isolation,
deferred work, and spec/plan scope. Omits Python typing. Full remains a standalone
superset of lite's guidance; the house libraries add stack-specific requirements.

## Status

**Complete.** Derived from a production app, with the drift judged case by case.
