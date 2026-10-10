---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: cmd/lyx/testdata/test-budget.yaml:4
    summary: cmd/lyx row stays at 73 although the branch adds TestRootHook_NoVerboseFlagKeepsTheLevelSetAtInit, so TestTestBudget_PackagesStayWithinBudget fails at 74 tests
---

### [BLOCKING] cmd/lyx test budget row not raised for the test the branch adds

**Location:** `cmd/lyx/testdata/test-budget.yaml:4` (row `cmd/lyx: 73`); the added test is `TestRootHook_NoVerboseFlagKeepsTheLevelSetAtInit` in `cmd/lyx/main_test.go`.
**Issue:** Commit `869feb6e9` (fix F1) added one top-level test to `cmd/lyx` and raised only the `internal/reedengine` row.
The merge-in commit `d51290bb5` brought main's `cmd/lyx` row, which was already at 73 with 73 tests, so the merged tree has 74 top-level tests against a budget of 73.
The Publish verify log (`.lyx/verify/verify-3.log`) reports `cmd/lyx: 74 top-level tests, over its budget of 73; raise its row in cmd/lyx/testdata/test-budget.yaml in the commit that adds the tests`.
`CLAUDE.md` requires the commit that adds the tests to raise the row.
Nothing else in the log fails: every other package passes, so this is the only cause of the Publish failure.
**Fix:** Change the row to `cmd/lyx: 74`.
No test is added or removed: the new test pins a behavior (the root hook keeps the init-time level when no `-v` is given) that no other test covers.

The rest of the range was read against the plan's cards (logger Debug admission, daemon spawn environment, watchdog stderr, told tmux binary, poll-mode backoff, socket-path refusal, hermetic `-f /dev/null` servers and shell pins, resize signal entry at boot) and showed no further defect.
