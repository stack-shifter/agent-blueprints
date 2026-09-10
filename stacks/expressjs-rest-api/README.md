# expressjs-rest-api

Vault-internal. Never copied out.

## Pick this when

You are working in a layered Express 5 REST API written in TypeScript — routers under a
version prefix, thin controllers, Zod validation middleware, repositories behind a database
context, and Cognito JWT authorization. Postgres through Drizzle ORM, deployed as a
container.

## Assumes

- Node 24, ESM, TypeScript, run as a long-lived process.
- Express 5 — automatic async error propagation, four-parameter error middleware, built-in
  body parsing.
- Zod 4 for request validation, applied as middleware that rejects but does not transform.
- Drizzle ORM with generated migrations checked in.
- The repository chooses its test runner and test layout.
- Resources are tenant-scoped under a parent path segment.

## Pick a different stack when

The API runs on Lambda behind API Gateway rather than as a container — that is
`aws-cdk-serverless-api`, which has a different bootstrap, a different error path, and
infrastructure in the same repository. This baseline assumes a process that boots once and
holds a connection pool.

## Where this disagrees with its reference project

Four places, all deliberate:

- **The reference project's own `AGENTS.md` and `CLAUDE.md` describe DynamoDB.** The code
  is Postgres through Drizzle — repositories, a `pg` pool, generated SQL migrations, and
  SQLSTATE constraint codes. The docs are inherited from a sibling project. The baseline
  follows the code.
- **The docs say the runtime depends on `PORT` and `CORS_ORIGIN`.** It reads a dozen more,
  including `DATABASE_URL`, the Cognito pool, an S3 bucket, SES identities, and a Bedrock
  model id. The baseline states the rule — document every variable in `.env.example` — and
  does not enumerate a list that will rot.
- **The bundled Express skill prescribes `res.locals.validated` and cursor pagination.**
  The code validates-then-re-reads from `req`, and offset pagination replaced cursors. The
  baseline follows the code, and says explicitly that moving to `res.locals` is an
  all-or-nothing change rather than a per-endpoint choice.
- **Middleware imports ordinary Zod builders from `zod/v4`.** The baseline normalizes
  application schemas to the package root and reserves `zod/v4/core` for shared internal
  types. That follows Zod's application guidance and avoids two import conventions for the
  same public API.

## Rules worth knowing about

- **Tenant scoping has its own section** because it is the only thing separating two
  tenants. Group-based authorization establishes who the caller is, never which workspace
  they may touch, and every handler must re-check ownership after loading by id.
- **Module-scope `process.env` reads are called out as a hazard.** Import timing differs
  among runners and deployment modes, so configuration is read in a function or at the
  composition root instead.

## Snippet subset

Carries twelve of the fourteen snippets. Omits `accessibility/wcag-aa-baseline` (no user
interface) and `typing/python-type-hints` (wrong language).

## Status

**Complete.** Derived from a working intake API.
