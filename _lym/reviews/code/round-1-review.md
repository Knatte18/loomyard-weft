---
verdict: APPROVED
---

Amendment round: the commit b1fe0b2d and the Parent-Align merge a7dfe2ea, reviewed against the focus questions.

- The new `TestRunCLI_BypassPushPointsTheTraceSinkAtItsHub` has no `t.Parallel()`, so Go finishes it before any paused parallel top-level test (`TestRunCLI_Anchor`, `TestRunCLI_HubMutationScenario`) resumes; the sink read cannot race them.
- It restores the one global the bypass branch repoints (`weft_verbs.go:265`) with `t.Cleanup(logger.SetDurableSinkDir(""))`.
- The scenario step keeps its exit-code, `RequireOK` and weft/warp SHA checks; only the sink assertion moved. No other test asserts `logger.TraceDir()` in `internal/fabriccli`, so nothing is duplicated.
- b1fe0b2d changes only `internal/fabriccli/hubmutation_integration_test.go`; no non-test code and nothing outside the board entry's scope.
- The parent merge brought in burlerengine, commentlint, loomcli, loomshed and shedrecipe changes; none touches `fabricengine`, `websterengine` or `planparser`, so it cannot contradict the settled scanner, recover-batch or reset behavior.

## Rejected

None.
