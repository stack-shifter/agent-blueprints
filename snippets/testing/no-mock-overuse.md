## Mocking

Mock at the system's edges, nowhere else.

- Mock only what you do not own: network calls, clocks, randomness, the filesystem, third-party services.
- Never mock the thing under test, and never mock your own internal modules to make a test easier to write. That couples the test to the implementation and it will pass while the code is broken.
- Prefer a real in-memory implementation (an actual sqlite, a fake repository) over a mock with scripted return values.
- If a test needs more than a couple of mocks to run, the design is telling you the unit has too many dependencies. Fix the design instead of adding mocks.
- Never assert that a mock was called as the primary assertion. Assert on the resulting state or output.
