# monorepo

Vault-internal notes. Not copied into projects — only `agent.md` is.

## Pick this when

The repository holds multiple packages under one workspace, regardless of language.

## How to use it

This baseline is cross-cutting rather than language-specific: it covers workspace layout,
task running, package boundaries, and release discipline. Start from it, then paste the
typing and style sections from the language baseline that matches the repo. The result is
one complete file, as always.

## Assumes

- A workspace manifest at the root
- A task runner that understands the package graph
- Generated version bumps, not hand-edited ones

## Status

Stub. Boundaries, toolchain, and release rules are written; language rules, testing, git,
and safety are TODOs.
