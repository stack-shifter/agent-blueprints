## Comments

Write comments that help a human follow the code: purpose, contracts, assumptions,
meaningful workflow stages, and rationale. Do not optimize for fewer comments.

- Use concise doc comments to explain a function's purpose and contract. Include
  the HTTP method and route for handlers or controllers where applicable.
- Use short inline comments to introduce meaningful logical steps, such as
  relying on validated input, delegating to a service, or protecting public error
  responses. Keep them even when a reader could infer the behavior from the code.
- Explain workarounds, non-obvious constraints, ordering requirements, and
  deliberate deviations from the normal pattern.
- Remove mechanical narration that adds no reading benefit, not useful summaries
  or workflow signposts. Correct inaccurate or stale comments when behavior changes.
- Never leave commented-out code. Version control already remembers it.
- No changelog comments, no attribution comments, no `// TODO` without a concrete
  next action or removal condition.
- Follow explicit project or user documentation preferences; otherwise match the
  closest existing peer in the same layer. Keep coverage uniform within a peer group.
- Document exported contracts: purpose, relevant inputs and outputs, assumptions,
  and meaningful errors. Use the language's documentation style without forcing
  redundant tags or narrating every statement.
