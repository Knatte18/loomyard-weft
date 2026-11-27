```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace the diff's changes to the board entry's goal points.

**Decisions.** These are the board entry's `## Goal` → `What to do` points. The record has no `Added after Discussion` entries.
- **G1**: record a failure with a non-empty transient class as a retryable stop, not as `failed` / `producer-hard-failure`, at least until the driver's re-steps are spent.
- **G2**: write no `loom-halt` note for such a stop, or one with its own anomaly kind.
- **G3**: log the parent-lock wait's start, end and holder in the trace.

**Per change**
- **C1**: `internal/loomengine/anomaly.go` adds `AnomalyTransientStop`, gives `ClassifyHalt` a `transient` parameter and adds the case that returns the new kind. Traces to G2 (the note's own kind). It also builds G1's "not `producer-hard-failure`" half.
- **C2**: `internal/loomcli/halt.go` adds `haltNote.Transient`, sets it in `failedHalt` from `st.Transient != ""`, and passes it in `writeHaltNote`. Traces to G2.
- **C3**: the `internal/loomcli/doc.go` paragraph on `transient-stop`. Forced by C1 and C2, because the package doc lists the halt kinds.
- **C4**: the `contracts/stencils/friction/friction-template-reflection.md` drop and file rule for `transient-stop`. Forced by C1: reflection must treat a new halt kind. It also serves the entry's "reaches reflection as a defect signal" complaint.
- **C5**: `internal/landingshed/parentguard.go` logs the wait's start with holder, each holder change, and the end with wall-clock and awake time. Traces to G3. The holder-change line and the awake field go a little past "start, end and holder", but they serve G3's stated aim, explaining a long wait. Not unrequested.
- **C6**: the tests in `internal/loomengine/anomaly_test.go` (the three new `TestClassifyHalt` rows and the five-argument call) and `internal/loomcli/halt_test.go` (the `GitHubAPIFailureMarkedTransient` row). Forced by C1 and C2.
- **C7**: commit fffd236d9, comment line breaks only. Forced by C1 and C3.

**Per decision**
- **G1**: partly built, by C1 and C2, and only on the halt note. No change alters what the status file records. See finding 1.
- **G2**: built by C1 and C2. C3, C4 and C6 are the edits they force.
- **G3**: built by C5.

No change traces to nothing. No commit in the range carries a `Pattern-Departure:` trailer.

**Findings**

1. BLOCKING · class `scope` · `internal/loomcli/halt.go:108` (`failedHalt`) and the shed persist path `internal/shedengine/run.go:406`.
   - **Summary:** G1 is not built. A transient-classed failure is still persisted as state `failed`. The diff changes only the halt note's anomaly kind, but G1 asks that such a failure be recorded as a retryable stop and not as `failed`.
   - **Evidence:**
     - `_lym/discussion/decision-record.md:45` asks for a retryable stop "not as `failed` / `producer-hard-failure`, at least until the driver's re-steps are spent".
     - `_lym/discussion/decision-record.md:35` names "the persisted state and the halt note" both as wrong.
     - `internal/shedengine/run.go:406` still persists `StateFailed` with the class, and no hunk in the diff touches it.
     - `_lym/landing/summary.md:20-23` keeps state `failed` on purpose ("no new shed state is added"). It presents this as a design decision of the writer's own, not as a correction recorded by the parent. The record has no `Added after Discussion` entry that allows it.
   - **Fix:** make the recorded state of a transient failure a retryable stop until the driver's re-steps are spent, so that the status file and the board stop reading `failed`. Or have the parent rule the narrower reading in scope; that ruling is not the fixer's to make.

I made no tool calls; the inherited context held everything this lens needed.
