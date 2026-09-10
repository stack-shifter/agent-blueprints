# aws-cdk-library

Vault-internal. Never copied out.

## Pick this when

You are working **on** a shared AWS CDK construct library — one that other repositories
install and build their stacks with.

## Assumes

- TypeScript, Node 24+, published as an npm package with a single entry point.
- Constructs wrap CDK L2s and expose them as public readonly fields.
- The repository chooses its test runner, test layout, and typecheck composition.
- Version comes from the release tag, not from a hand-edited `package.json`.

## Pick the sibling instead when

You are **using** such a library to build an API — writing endpoints, handlers, and
tables. That is [`aws-cdk-serverless-api`](../aws-cdk-serverless-api/).

The distinction that matters: here, every public name is a contract and a rename replaces
someone else's deployed infrastructure. There, you consume those names and the risk lives
in the deploy.

## Snippet subset

Carries twelve of the fourteen snippets. It omits `accessibility/wcag-aa-baseline` because
the library has no user interface, and `typing/python-type-hints` because it is TypeScript.

## Status

**Complete.** Derived from a production construct library.
