---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/orchengine/state_test.go:85
    summary: the new TestPeekState was inserted between the testtiming:keep directive and TestCycleRequest_WriteProbeClear, so the directive now annotates the wrong test
    fix: in internal/orchengine/state_test.go move the line "//testtiming:keep pins the write, probe and clear round trip of the cycle request marker, which its covering test does not assert" from above TestPeekState to directly above func TestCycleRequest_WriteProbeClear, leaving TestPeekState with no directive.
---

### [MEDIUM] testtiming:keep directive detached from its test

**Location:** internal/orchengine/state_test.go:85
**Issue:** The diff adds `TestPeekState` immediately after the `//testtiming:keep pins the write, probe and clear round trip of the cycle request marker ...` line.
That directive described `TestCycleRequest_WriteProbeClear`, which now sits below `TestPeekState` with no directive, while `TestPeekState` carries a keep reason about the cycle request marker.
`cmd/testtiming` reads the directive as belonging to the function it directly precedes, so the keep exemption is lost for the cycle-request test and falsely claimed for `TestPeekState`.
`lyx gate test` on both packages passes, so this is not caught by the test run.

## Rejected

- PeekState is lockless and creates nothing: the body is `os.ReadFile` plus `json.Unmarshal`, imports stay stdlib, and `TestPeekState` pins the no-lock-file and no-mkdir halves. A partial or mid-write file cannot occur because `internal/state` writes atomically, and a half-decoded file would return a wrapped error.
- The orch poll applies the run lines' two-strike tolerance, with the repeat-at-same-modification-time rule and a stat failure counted as a failure; thresholds load once in `overviewCmd`; `watcher down` follows `WatcherExit`, which `watchloop.go` and `watcher.go` set at exit and `state.go` clears at start.
- The diff leaves `lyx orch status`, the watcher, the orch state fields and the orch config unchanged; the only `orchengine` changes are `PeekState` and a `doc.go` sentence.
- Test economy: `TestPeekState`, the new `TestRenderOverview` rows, `TestOverviewSource_DrawsTheOrchLine` and `TestLoadOverviewThresholds` each pin a behavior no existing test covers, and the budget rows rise by the new top-level tests (one and two).
