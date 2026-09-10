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

The reference application is on React Router 7. This baseline deliberately targets version
8 so a new copy uses the current module boundaries and no removed compatibility package.

## Snippet subset

Carries thirteen of the fourteen snippets. It omits `typing/python-type-hints` because this
is a TypeScript stack.

## Status

**Complete.** Derived from a production app, with the drift judged case by case.
