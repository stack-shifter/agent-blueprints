# angular-spa

Vault-internal. Never copied out.

## Pick this when

You are building or maintaining an Angular single-page application — standalone
components, signals, reactive forms, lazy-loaded routes.

## Assumes

- Angular 21, TypeScript, npm, the Angular CLI.
- Standalone throughout; no NgModules.
- Signal-based component state, `OnPush` everywhere, native template control flow.
- The repository chooses its unit-test runner and test layout; Angular's functional HTTP
  testing providers remain the stack-level testing API.
- A component library and a hosted-auth provider are likely present but not required —
  no rule here depends on which.

## Pick a sibling instead when

You are building a React SPA. That is [`react-vite-spa-lite`](../react-vite-spa-lite/) if
the project brings its own auth, styling and forms, or
[`react-vite-spa-full`](../react-vite-spa-full/) for the house set.

Unlike the React pair, this is a single baseline. Angular ships forms, DI, routing and HTTP
in the framework, so the variable surface is thin enough that a lite/full split would
duplicate almost everything to isolate two sections.

## Note on prescription

This baseline deliberately disagrees with the app it was derived from, and with that app's
own guidance file, in named places.

It **mandates** `strict` plus an `angularCompilerOptions` block with `strictTemplates` —
the reference app has neither, which is what lets `input<T>(undefined)` compile and what
hides a real crash on an unassigned dialog reference.

On one further point it **sides with the framework over the codebase**. That app's
`AGENTS.md` is Angular's official LLM guidance file rather than anything hand-written, and
the codebase never uses the async pipe — observables are drained into signals or
subscribed by hand. The baseline restores the framework position: read observables in the
template with the `async` pipe. Expect it to disagree with an existing Angular codebase
there.

Where the framework advises by size, the baseline does not. Angular's file prefers inline
templates for small components; this baseline requires `templateUrl` unconditionally,
which is what the reference codebase does for all of its components including one-line
ones. A rule that changes with a file's length is a rule nobody applies consistently.

## Snippet subset

Carries thirteen of the fourteen snippets. It omits `typing/python-type-hints` because this
is a TypeScript stack.

## Status

**Complete.** Derived from a production app, with the drift judged case by case.
