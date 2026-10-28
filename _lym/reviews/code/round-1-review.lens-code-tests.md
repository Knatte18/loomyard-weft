```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it**

- **A bypass push from a hub's board worktree points the trace sink at that hub's logs directory.** Pinned by `TestRunCLI_BypassPushPointsTheTraceSinkAtItsHub` (`hubmutation_integration_test.go:31-48`). It asserts `logger.TraceDir() == fabricengine.HubLogsDir(h.Path)` after a push that exits 0. If `weft_verbs.go:265` stopped repointing the sink, the assertion would fail.
- **A bypass push pushes both sides.** Still pinned by the `BypassPushAdvancesBothUpstreams` step of `TestRunCLI_HubMutationScenario`. It keeps the exit-code check, `envelope.RequireOK`, and the weft and warp bare SHA assertions. Only the sink assertion left the step.
- **The sink is restored.** Both the new test and the scenario step register `t.Cleanup(logger.SetDurableSinkDir(""))`.
- **The new test is race-free.** It has no `t.Parallel()`, so Go runs it to completion before any parallel test is resumed. The parallel tests, such as `TestRunCLI_Anchor` at `anchor_integration_test.go:52` and the scenario, therefore cannot repoint the sink between the push and the read.
- **No duplicated coverage.** `TraceDir()` is asserted nowhere else in `internal/fabriccli` tests. The anchor tests only reset the sink in cleanup.

The parent merge `a7dfe2ea` brought in burlerengine, commentlint, loomcli, loomshed and shedrecipe changes. They are outside this lens and the amended scope, and they carry their own test files in the same commits.

**Findings:** none.

**Tool calls (2)**
1. A grep of `TraceDir()` in the fabriccli tests, plus the test names in `anchor_integration_test.go`. I needed it to check whether another test already covers the sink.
2. `sed -n 25,40p` of `anchor_integration_test.go`. I needed it to see how the anchor hub helper handles the sink.
