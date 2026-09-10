# aws-cdk-serverless-api

Vault-internal. Never copied out.

## Pick this when

You are building or maintaining a production serverless HTTP API on AWS — API Gateway and
Lambda — with CDK, composed from a shared construct library. The datastore may be DynamoDB
or SQL; the baseline covers both and marks which rules depend on the choice.

## Assumes

- TypeScript, Node 24+, CDK app with `bin/`, `lib/`, and a layered `src/`.
- Routes composed through the shared library's `RestApi`, not hand-rolled API Gateway
  wiring; IAM granted with callbacks.
- Middleware-composed Lambda handlers and schema validation at the boundary.
- A datastore reached through repositories — single-table DynamoDB, or SQL with committed
  migrations and a pooled connection budget.
- Cognito authorizers at the gateway; `authorizer: null` for a public route.

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

Carries twelve of the fourteen snippets. It omits `accessibility/wcag-aa-baseline` because
the API has no user interface, and `typing/python-type-hints` because it is TypeScript.

## Status

**Complete.** Derived from a production API, with the drift corrected.
