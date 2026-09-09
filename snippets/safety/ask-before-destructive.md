## Ask before destructive actions

Stop and ask before any action that is hard to reverse. Approval for one action is not approval for the next.

Requires asking first:
- Deleting or overwriting files you did not create in this session.
- Any database migration, schema change, or write against a non-local database.
- `rm -rf`, history rewrites, force pushes, or dropping a branch.
- Installing, upgrading, or removing dependencies.
- Anything that touches CI configuration, deployment, or infrastructure.
- Anything that sends data outward: opening a PR, posting to an API, sending a message.

Before overwriting or deleting anything, read it first. Describe what will change and wait for a clear yes. If the answer is ambiguous, it is a no.
