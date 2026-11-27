```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I named the test that pins it, or none.

**Per behaviour, the test that pins it**

- **A failed state with transient set gives `AnomalyTransientStop`** (`internal/loomengine/anomaly.go:116`): pinned by `TestClassifyHalt/FailedTransient_TransientStop` (`internal/loomengine/anomaly_test.go:170`).
- **An interrupted stop with transient set stays `AnomalyInterrupted`** (`anomaly.go:112`, case order): pinned by `TestClassifyHalt/FailedInterruptedTransient_InterruptedStop` (`anomaly_test.go:178`).
- **A non-failed state with transient set reports no kind** (`anomaly.go:120`): pinned by `TestClassifyHalt/NotFailedTransient_NoKind` (`anomaly_test.go:186`).
- **A failed state with neither flag stays `AnomalyProducerFailure`**: pinned by `TestClassifyHalt/FailedArbitraryError_ProducerHardFailure` (`anomaly_test.go:156`) and `TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection/ProducerFailure` (`internal/loomcli/halt_test.go:477`).
- **buildSkew wins over transient when both are set** (`anomaly.go:114` comes before `:116`; the doc comment at `anomaly.go:94-96` states this order): no test. No row sets both `buildSkew` and `transient`, so swapping the two cases passes every test.
- **`failedHalt` reads `st.Transient != ""` into `haltNote.Transient`** (`internal/loomcli/halt.go:127`), which `writeHaltNote` passes to `ClassifyHalt` (`halt.go:74`): pinned by `TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection/GitHubAPIFailureMarkedTransient` (`halt_test.go:484-489`). The producer's mark survives the persist, through `MarkTransient`/`TransientOf` at `internal/shedengine/run.go:402-406`. If the field were dropped, the note would read `producer-hard-failure` and the row would fail. The same `failedHalt` serves the arm path (`internal/loomcli/arm.go:569`), so that path is covered too.
- **The parent-lock wait logs start, holder change and end with wall-clock and awake time** (`internal/landingshed/parentguard.go:62-84`): none. These are trace log lines that only people read, and testing rule "Test what something relies on" exempts them. The summary says so at `_lym/landing/summary.md:31`. No finding.
- **The reflection template's `transient-stop` rule** (`contracts/stencils/friction/friction-template-reflection.md:51-53`): stencil prose, exempt.

**Findings**

1. **LOW, consistency**, `internal/loomengine/anomaly_test.go:170`: `TestClassifyHalt` has no row with both `buildSkew` and `transient` set. The precedence that `ClassifyHalt`'s doc comment states (`anomaly.go:94-96`) and its case order encodes (`anomaly.go:114-117`) is therefore unpinned. Fix: add a row `FailedBuildSkewTransient_BuildSkew` with `state: StateFailed`, `buildSkew: true`, `transient: true` and `wantKind: AnomalyBuildSkew`.

Tool calls:
- Read `internal/loomengine/anomaly_test.go:125-170` and grepped `internal/loomcli` tests for `Transient`, to check the existing `ClassifyHalt` rows and any other loomcli transient test. My context held only the diff hunks, not the unchanged rows.
