## Comments

Comment why, never what. If a comment restates the code, delete the comment or fix the name.

- Write a comment when the reason for the code is not recoverable from the code: a workaround, a non-obvious constraint, an ordering that matters, a deliberate deviation.
- Never leave commented-out code. Version control already remembers it.
- No changelog comments, no attribution comments, no `// TODO` without a concrete next action.
- Match the comment density of the surrounding file. A heavily annotated codebase and a bare one both have a house style; follow it.
- Doc comments on exported symbols state contract, not implementation: what it takes, what it returns, what it throws, what it assumes.
