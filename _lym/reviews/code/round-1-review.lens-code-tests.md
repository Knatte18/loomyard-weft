```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: list each behaviour the diff adds or changes, with the test that pins it.

Behaviours the diff adds or changes, each with the test that pins it:

- None. `git diff 3fb0787e32473e6bd8d961da278a4bcecb4d2460..HEAD` prints 0 lines, and HEAD is the review base `3fb0787e3`. The range has no commits and no hunks, so no behaviour was added or changed that a test would need to pin.

Findings: none.

The change description says the base behaviour is pinned by `TestResetCmd_RecordsTheRunTransitionAtTheNewTip` (`internal/webstercli/reset_test.go:171`). That code is outside the diff. I read the test anyway:
- Its `UnreachableRecordsOriginLeavesAnOwedRecord` row checks both `push_error` and `FailedPushesOwedBy` (`reset_test.go:208-213`).
- The other rows check that origin's records branch matches the records HEAD (`reset_test.go:214-220`).

So the claim holds at the level my lens covers. No test exercises the base path where `fabricSync` returns a partial-commit error after the weft commit landed (`internal/fabricengine/commit.go:154-176`, `reset.go:193-195`). That path is not part of the diff, so it is outside this lens; I'm noting it for the handler only.

Tool calls:
- `git diff … | wc -l; git rev-parse HEAD`: confirmed the range is empty. My context had only the `--stat` output, which printed nothing, and lacked an explicit check of the hunk count and HEAD.
