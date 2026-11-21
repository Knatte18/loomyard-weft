```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that would fail if it broke.

**Behaviours and the tests that pin them**

1. **`renderFailureReport` / `failedRun.reportLines` print a run's failures in order, with no budget pass for triage lines.** Pinned by `internal/gatecli.TestRenderFailureReport`. Its table rows compare the full output exactly with `cmp.Diff` (`internal/gatecli/failurereport_test.go:138`), and its line-cap subtest checks the cut and the line budget. If the loop or the budget broke, a row would fail. Covered.
2. **`execution.printReport` no longer adds triage lines before printing.** Pinned by `internal/gatecli.TestGateTest_Scenario`, subtest "a real go test tier's failed test and broken package are both in the failure report…" (`internal/gatecli/test_integration_test.go:1434-1464`). It checks the header, the run line, the test line, the output line and the touched files. The reach report's "no mapped test reaches `printReport`" note is explained by this test running out of process. Covered for what printReport still does.
3. **`gateslot.FailedTests` / `TierRunFailures` read failure reports that no longer carry triage lines.** Pinned by the remaining `TestFailedTests` rows and by `TestTierRunFailures`. The parser itself did not change. Covered.
4. **loomshed `verifyFailureFindings` no longer adds a `Triage:` section.** Pinned by `TestVerifyFailureFindings`: its rows check the Failures section, the log tail and the watchdog text. No row checks that `Triage:` is absent, but nothing that renders one remains in the tree, so there is nothing to regress. Covered.
5. **`runplan.RenderVerifyGateFindings` no longer writes a triage section in the hang branch or the identities branch.** Pinned by `internal/websterengine.TestRenderVerifyGateFindings` and the verify-gate tests the reach report lists (`TestVerifyGate_FailedEvaluationWritesReport` and others). Covered.
6. **`runplan.VerifyGateReport` drops its `Triage` field, and weaver's `failureReport` and webster's `failHung` and rerun report no longer fill it.** Pinned by `TestVerifyGate_FailedEvaluationWritesReport`, which keeps its checks on attempt, cap, failures, hint, fix commits and log path, and by weaver's `TestRun` and `TestRun_Reentry` (reach report). The deleted weaver `TestFailureReport` only checked `.Triage`, so no other coverage went with it. Covered.
7. **An older `verify-gate.yaml` that still holds a `triage:` key keeps decoding.** The change description claims this, and it holds because `readVerifyGateReport` uses plain `yaml.Unmarshal` (`internal/websterengine/verifygate.go:526-536`). No test pins it, but this is tolerance the code already had under PATTERN-tolerant-records, not behaviour the diff adds. No finding.

**Removed tests**

Each covered triage behaviour only:
- `TestTriageLookup_Lines`
- `TestTriageLines`
- the triage row in `TestFailedTests`
- the triage row and cap subtest in `TestRenderFailureReport`
- the mapped-test integration subtest, and the triage assertion plus prime `go.mod` setup in the report subtest
- weaver's `TestFailureReport`
- the Triage checks in `TestVerifyFailureFindings`, `TestRenderVerifyGateFindings` and `TestVerifyGate_FailedEvaluationWritesReport`

No surviving assertion was loosened.

**Findings**

None. The diff only removes behaviour, and every behaviour that remains in the changed functions has a test that would fail if it broke.

Tool calls:
- `sed -n 134,150p internal/gatecli/failurereport_test.go`: to confirm that the `TestRenderFailureReport` rows compare the full output with `cmp.Diff`. The diff piece showed only the rows, not the comparison loop.
