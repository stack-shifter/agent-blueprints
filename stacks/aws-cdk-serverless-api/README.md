# aws-cdk-serverless-api

Vault-internal. Never copied out.

## Pick this when

You are building or maintaining a production serverless HTTP API on AWS — API Gateway and
Lambda — with CDK, extending installed libraries or local constructs. The datastore
may be DynamoDB or SQL; the baseline marks which rules depend on the choice.

## Assumes

- TypeScript, Node, AWS CDK, and layered runtime responsibilities; installed
  configuration determines versions and interfaces. Preserve existing folder layout.
- Extend installed construct libraries or existing local constructs. Verify exports,
  default merging, authorizer overrides, and runtime/infrastructure import boundaries.
- HTTP handlers compose middleware; asynchronous workers delegate to services.
- Validation deliberately passes parsed schema output to controllers.
- Repositories own persistence. Preserve DynamoDB access patterns or SQL constraints,
  committed migrations, narrowly justified locks, and connection capacity.
- Tests isolate configuration and external calls. SQL integration tests use a
  disposable database; bundle checks complement infrastructure template assertions.

## Pick the sibling instead when

You are working on the construct library itself rather than an API that consumes it. That
is [`aws-cdk-library`](../aws-cdk-library/).

## Note on prescription

This baseline is deliberately stricter than the production service it was derived from.
It prescribes `instanceof` error routing, CORS headers on middleware responses, validated
startup configuration instead of non-null assertions, and template assertions for `lib/` —
each of which corrects a real gap in the reference implementation. Expect it to disagree
with an existing codebase in those specific places; that disagreement is the point.

## Snippet subset

Carries sixteen snippets, including scoped authorization, test isolation, deferred
work, and spec/plan scope. Omits both accessibility variants because this API has no
interface, Python typing because it uses TypeScript, and repeated-approval guidance
in favor of approved scope. Shared wording is pasted in full and remains standalone.

All stacks carry the canonical README consistency standard in full.

## Status

**Complete.** Derived from a production API, with the drift corrected.
