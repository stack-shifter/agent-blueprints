# react-vite-spa-lite

Vault-internal. Never copied out.

## Pick this when

You are building a React single-page application and bringing your own choices for auth,
styling, forms, and data fetching — or joining a project that already made them.

## Assumes

- React 19, Vite, TypeScript strict, npm. The baseline names its assumed majors in
  one place and says the project wins on a mismatch.
- React Router 8 data router (`createBrowserRouter` and `RouterProvider` from
  `react-router/dom`; everything else from `react-router`).
- The repository chooses its test runner, component-testing library, and test layout.
- Nothing about how you authenticate, style, validate, or fetch.

## Pick the sibling instead when

You want the house set decided for you — Cognito hosted-UI auth, Bootstrap, React Hook
Form with Zod. That is [`react-vite-spa-full`](../react-vite-spa-full/).

The two are **peers, not layers**. `full` is a standalone superset that repeats this
file's text; it is not an overlay to paste on top of this one.

The reference application is on React Router 7. This baseline deliberately targets version
8 so a new copy uses the current module boundaries and no removed compatibility package.

## Snippet subset

Carries thirteen of the fourteen snippets. It omits `typing/python-type-hints` because this
is a TypeScript stack.

## Status

**Complete.**
