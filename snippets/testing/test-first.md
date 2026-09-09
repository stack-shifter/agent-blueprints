## Test discipline

A bug fix starts with a test that reproduces the bug and fails. A feature starts with a test that describes the behavior.

- Write the failing test first, watch it fail for the right reason, then make it pass. A test that has never failed has not been verified.
- One behavior per test. The test name states the behavior in a sentence: `returns_empty_list_when_no_matches`, not `test_search_2`.
- Assert on observable behavior, not on internal calls. A test that breaks on a refactor with no behavior change is a bad test.
- Cover the boundaries: empty, one, many, and the error path. The happy path alone is not coverage.
- Never weaken an assertion or delete a test to make a suite green. If a test is wrong, say so explicitly and explain why before changing it.
- Never report work as done on an unrun suite. If tests cannot be run, say that plainly rather than implying they passed.
