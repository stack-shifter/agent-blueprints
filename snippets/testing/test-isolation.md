## Test isolation

- Set deterministic test configuration before importing application modules. Never
  inherit developer service URLs, auth sessions, database URLs, or credentials.
- Stub external boundaries and block unexpected outbound calls, including failures
  caught by application code. Allow only explicitly configured disposable test
  services or deterministic stubs.
- Assert expected logs locally and fail on unexpected warnings/errors. Never mute
  the entire suite to hide missing fixtures or genuine failures.
- Keep automated tests independent of live services. Report explicitly authorized
  live verification separately; it does not replace deterministic tests.
- Follow existing test locations, naming, and shared setup. Do not reorganize tests
  or introduce another runner as incidental feature work.
