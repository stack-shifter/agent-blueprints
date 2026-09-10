# aspnetcore-rest-api

Vault-internal. Never copied out.

## Pick this when

You are working in a layered ASP.NET Core REST API written in C# — attribute-routed
controllers under a version prefix, DTOs validated by data annotations, repositories behind
one aggregate interface, Entity Framework Core over Postgres, and JWT bearer authorization.
Deployed as a container.

## Assumes

- .NET 10, C# 14, nullable reference types enabled, implicit usings on.
- EF Core with generated migrations committed to the repository.
- Controllers use primary constructors and `[ApiController]` model validation.
- Offset pagination with totals, not cursors.
- The repository chooses its test runner, mocking library, test layout, and integration-
  database strategy.

## Pick a different stack when

The API is Node rather than .NET — that is `expressjs-rest-api`, which has the same layered
shape but a different toolchain, validation library, and ORM. If it runs on Lambda behind
API Gateway rather than as a long-lived container, that is `aws-cdk-serverless-api`.

## Why this one prescribes harder than its siblings

Every other baseline resolved a disagreement between a project's docs and its code by
siding with the code. Here the **code disagrees with itself**: four sibling repositories
implement the same interface four different ways — one fetches by id with `FindAsync`,
another with `AsNoTracking().Include(...).FirstOrDefaultAsync`, a third with
`SingleOrDefaultAsync`. So this file picks one variant per decision and states it, rather
than describing a house style that does not exist yet.

## Rules that exist because of a real defect

Two rules correct live bugs in the reference project, and one guards a validation failure
that is easy to miss in review:

- **Order by a key set ending in the primary key.** Three of the four repositories page on
  a non-unique sort column, so rows repeat or vanish as a caller pages. It is invisible
  until someone reads page two.
- **Use the built-in `RequiredAttribute`.** The current DTOs import it correctly. The rule
  guards against an unrelated, identically named attribute that compiles but does not derive
  from `ValidationAttribute`, so ASP.NET Core model validation does not enforce it.
- **Mappers are pure.** One mapper calls the object store to sign a URL inside its `ToDto`,
  which the controller then calls once per item — a page of 200 signs 200 URLs.

The empty-collection rule has the same origin: the controllers return `404` when a filtered
list comes back empty, which tells a caller the endpoint does not exist rather than that
nothing matched.

## Note on the snippet subset

Carries ten of the fourteen snippets. It omits:

- `accessibility/wcag-aa-baseline` — no user interface.
- `typing/strict-typescript` and `typing/python-type-hints` — wrong languages. The C#
  nullability rules live inline in `## Models and nullability` instead. If a second .NET
  baseline ever appears, promote that section to `snippets/typing/nullable-csharp.md` and
  paste it into both.
- `style/naming-conventions` — its file-naming bullet names `.ts` files, which would paste
  a TypeScript instruction into a C# baseline. `## Naming` is written for C# instead and is
  deliberately not canonical.

## What the baseline ignores on purpose

The reference project carries a long design document proposing a serverless rewrite —
TypeScript CDK, DynamoDB, a search index, a new version prefix. None of it is implemented,
and the document never states its own status. It describes a different stack, so nothing
from it reached this baseline.

It also carries a real-time hub class that is never registered or mapped, so it cannot
receive a connection. That is dead code in one repository, not a rule for the stack.

## Status

**Complete.** Derived from a production logistics API.
