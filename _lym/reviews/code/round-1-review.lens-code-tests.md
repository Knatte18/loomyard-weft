```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that fails when that behaviour breaks.

**Per behaviour, the test that pins it**

1. **Each step's `run` split into `judgeReport` and `writeReport`** (reach, redundancy, mutation): pinned. The existing rows of `TestGateTestReachStep`, `TestRedundancyStep` and `TestGateMutate` now call `writeReport(judgeReport(...))`. The redundancy kept-report branch of `writeReport` is pinned by the `again.Written.Equal(kept.Written)` row at `internal/gatecli/redundancy_integration_test.go:446`.
2. **A judged report reaches the gate directory only on `write`, carries the gate key, and prints its lines in step order**: pinned by `TestReportChain`, row `AWaitedChainWritesItsReportsInStepOrder` (`internal/gatecli/reports_test.go:770-801`).
3. **Step lines and memory-guard lines are held in buffers while the steps judge, then flushed before each report's line**: none. Every row runs with `inHub=false`, where `judgeReport` returns `noteNoHub` and prints nothing, and with `guarding` unset. The buffers in `flushLines` are always empty.
4. **A stopped chain writes nothing**: none beyond row 2. `AStoppedChainWritesNothing` calls `wait()` before `stop()`, so the steps have already ended and no cancellation is exercised. Its only extra assertion, empty stderr, holds whatever the code does.
5. **The chain starts beside the first run of the plan's highest tier (`lastTierStart`)**: none. The one runTest-level row (`test_integration_test.go:1539`) has a single-tier plan, so any start index of 0 passes. The reach report lists `lastTierStart` as reached by no test.
6. **A run that fails after the start cancels the chain, and the call judges red-gate reports**: pinned by the `ARedTier` row at `internal/gatecli/test_integration_test.go:1551`, which checks `report.RedGate == true` and exit 7.
7. **A green call's chain hands over its report with the gate key**: pinned by the `AGreenTier` row at `internal/gatecli/test_integration_test.go:1550`.
8. **`reportStepsOf` maps the key's `reports` to steps, and `any`**: partly. Only `ReportRedundancy` reaches runTest in a test. The reach and mutation mapping is unpinned, as it was before the diff.
9. **A halted call writes no report and waits for the chain (deferred `stop`)**: none. No halt row has a key with `reports`.
10. **`withoutForeign` leaves a test binary below a foreign pid alone**: pinned by the new `TestMemoryGuard` row "a test binary below a running tier command is left to the tier's own guard" (`internal/gatecli/memoryguard_test.go:164-174`). Without the filter `wantKilled` would also hold `otherPID`.
11. **`runningCommands` add, remove and holds**: `add` and `holds` are pinned through that same row. `remove` is pinned by nothing.
12. **Production wiring of the foreign set**: none. This covers `request.commands = &runningCommands{}` (`run.go:814`), `request.commands.add`/`remove` in `runInvocations` (`tierrun.go:532-558`) and `guard.foreign = j.tierCommands.holds` (`reports.go:609`). Dropping any of them leaves every test green.

**Findings**

- **MEDIUM**, class `design`, location `internal/gatecli/reports.go:556` (`lastTierStart`) and `internal/gatecli/run.go:836`. Where the chain starts in a multi-run plan has no test. The only runTest row is single-tier (`internal/gatecli/test_integration_test.go:1539`), so a wrong index (always 0, or the lowest tier) passes every test.
- **MEDIUM**, class `design`, location `internal/gatecli/reports.go:567-645` (`judgedReports` buffers and `flushLines`). No test pins that the step and guard lines are held until `write` and print in step order. `TestReportChain` judges with `inHub=false` and the guard off, so its buffers are always empty (`internal/gatecli/reports_test.go:768`). A step writing straight to the call's stderr while it judges would pass.
- **MEDIUM**, class `design`, location `internal/gatecli/reports.go:609`, `internal/gatecli/tierrun.go:532-558` and `internal/gatecli/run.go:814`. Nothing tests that the report steps' guard actually spares the running tier's test binaries. `TestMemoryGuard` builds the foreign set by hand (`internal/gatecli/memoryguard_test.go:183-187`), so dropping `guard.foreign = ...`, `request.commands.add` or `request.commands.remove` fails no test.
- **LOW**, class `design`, location `internal/gatecli/reports_test.go:760` (row `AStoppedChainWritesNothing`). The row adds nothing to the other row. It stops the chain only after `wait()` returned, so no cancellation runs. Its "stat before write" check repeats the `AWaitedChainWritesItsReportsInStepOrder` row, and its empty-stderr check holds whatever the code does. Fix: fold it into the other row, or make it cancel a chain whose steps are still judging.
- **LOW**, class `design`, location `internal/gatecli/run.go:830-834` and `run.go:849-854`. No test pins that a halted call with a `reports` key writes no report and returns only once the chain has ended.

Tool calls:
1. Read the renamed redundancy scenario subtest in `internal/gatecli/test_integration_test.go` and grepped the package's tests for report-step and red-gate use. My context had only the subtest's name change from the diff, not its body or the other report-step tests.
