```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each diff change to the decision record's goal points and its `Added after Discussion` entry.

**Decisions**
- G1. Goal: "detect the build skew when the driver steps between segments and halting as a resumable state, not `failed`". Built by no change. A1 overrides it, so this is no finding.
- G2. Goal, alternative: "have the driver re-exec itself on the current binary". Built by no change. A1 records that the existing hand-over already does this, so this is no finding.
- G3. Goal, alternative: "surface the skew in `lym loom` status before a seat needs the stencil". Built by no change. A1 rules out a pre-step hand-over, option (c), so this is no finding.
- A1. Added after Discussion (parent, 2026-11-23), "Narrow to the false hard-failure report", option (b): a stop the driver hands over to a fresh build is not filed as `producer-hard-failure`. It gets no halt note, or a distinct anomaly kind that the selfreport path does not turn into an issue. Built by C1, C2 and C3.

**Changes**
- C1. `internal/loomengine/anomaly.go` adds `AnomalyBuildSkew` (`build-skew`) and a `buildSkew` parameter on `ClassifyHalt`, with a new case for a failed state with skew. Traces to A1: the distinct anomaly kind.
- C2. `internal/loomcli/halt.go` adds the `haltNote.BuildSkew` field and has `failedHalt` set it through `errors.As` for `*stencilstore.NewerBoardCopyError`. `writeHaltNote` passes it on. Traces to A1: it stops the hand-over stop's note being filed as `producer-hard-failure`.
- C3. `contracts/stencils/friction/friction-template-reflection.md` adds the rule that a `build-skew` halt is dropped. Traces to A1: the selfreport path does not turn the new kind into an issue.
- C4. `internal/loomcli/doc.go` gets a package-doc line naming `build-skew`. Forced by C1 and C2: the package doc already names the anomaly kinds of halt notes.
- C5. `internal/loomcli/halt_test.go` makes `TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection` table-driven and adds a row with a wrapped `NewerBoardCopyError`. Forced by C2.
- C6. `internal/loomengine/anomaly_test.go` adds the `buildSkew` column and two rows to `TestClassifyHalt`. Forced by C1.

**Findings**

1. Severity LOW, class `scope`. Location: `internal/loomcli/halt.go:116-123`, `failedHalt`.
   The diff labels every failed halt whose error chain holds `*stencilstore.NewerBoardCopyError` as `build-skew`, including halts the driver hands back rather than over.
   A1 licenses the distinct kind only for "a stop the driver hands over to a fresh build". `failedHalt` classifies from the error alone and never reads the driver's decision. When the on-disk build is unchanged, `decideDrive` does not take the hand-over row, and the stencil rule then drops that stop as well.
   The change description admits this widening in its Design decisions ("When the on-disk binary is no newer than the board, the driver hands the run back instead of over … carries `build-skew` there too"). No `Added after Discussion` entry or `Pattern-Departure:` trailer licenses it.
   Evidence:
   - `internal/sheddriver/driverules.go:264-265`: the hand-over row needs `facts.BuildChanged && !facts.HandedOver`.
   - `internal/loomcli/halt.go:116,123`: `BuildSkew: errors.As(err, &skew)`, with no condition on the hand-over.
   - `_lym/discussion/decision-record.md`, the added entry's Decision line.
   - `_lym/landing/summary.md`, Design decisions, second bullet.
   Fix: either have the parent record the wider classification with `lym loom decision add`, or narrow the claim. As a `decision`-flavoured gap, it is deferred to the parent rather than reverted.

No other finding. Every change traces to A1 or is forced by one that does, and every decision is either built or superseded by A1.

Tool calls: none. My inherited context already held the decision record, the summary, the diff pieces and `driverules.go`.
