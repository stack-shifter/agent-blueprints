# html-vite

Vault-internal. Never copied out.

## Pick this when

You are building a site or browser interface with authored HTML, plain CSS, and
vanilla JavaScript modules, using Vite for development and production builds.
Use it for either a single entry page or a multi-page project.

## Assumes

- Vite and browser-native HTML, CSS, and JavaScript; no UI framework or router.
- The manifest, lockfile, and Vite configuration determine versions, package
  manager, entry pages, asset paths, and deployment base.
- Preserve existing styling, authentication, and test-tool choices.
- Verify static changes in the browser and test meaningful JavaScript behavior
  with the project's existing tools. Report missing tools and unverified checks.
- Client-exposed configuration is public; secrets require a backend boundary.
- WCAG conformance applies only when the project specifies a version and level.

## Pick a sibling instead when

- Choose [`react-vite-spa-lite`](../react-vite-spa-lite/) for a React application
  with project-selected libraries, or [`react-vite-spa-full`](../react-vite-spa-full/)
  for the house set of libraries.
- Choose [`prototype-docs`](../prototype-docs/) for a product documentation vault
  with self-contained HTML prototypes rather than a Vite-built site.

These are standalone peers. Copy one baseline, then customize the destination.

## Snippet subset

Carries fifteen canonical snippets: project accessibility requirements; boring
code, vertical layout, naming, comments, and deferred work; mock boundaries, test
isolation, and verification; branch discipline and commit messages; scoped
authorization and secrets; spec/plan scope; and README consistency.

Omits both typing snippets because the stack uses JavaScript. Omits the blanket
test-discipline snippet in favor of behavior tests and browser checks appropriate
to static markup and CSS. Omits unconditional WCAG and ask-before-destructive
guidance in favor of project requirements and scoped authorization. Shared
wording is pasted in full; the baseline depends on no vault paths.

## References

- [Vite guide](https://vite.dev/guide/)
- [Vite asset handling](https://vite.dev/guide/assets)
- [Vite environment variables](https://vite.dev/guide/env-and-mode)

## Status

**Complete.**
