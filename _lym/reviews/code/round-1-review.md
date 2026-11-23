---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/friction/friction-template-reflection.md:48
    summary: The build-skew reason in the reflection stencil, the loomcli package doc and the AnomalyBuildSkew doc misstates what the driver does when it does not hand the run over.
    fix: "Reword all three sites to match the driver's rule table. In contracts/stencils/friction/friction-template-reflection.md:48-49, replace the reason after the colon with: the run's lym was older than a board stencil it read, which is a binary to update and not a lym defect; the driver hands such a run to a fresh driver when a newer build is on disk. In internal/loomcli/doc.go:13-14, make the same change to the clause after the colon. In internal/loomengine/anomaly.go:31-32, drop \"or back to its parent to update the binary\" and state that the driver hands such a run to a fresh driver when a newer build is on disk, so no producer is at fault."
  - id: F2
    severity: LOW
    class: decision
    origin: lens:code-scope
    location: internal/loomcli/halt.go#failedHalt
    summary: failedHalt marks every failed halt with a NewerBoardCopyError in its chain as build-skew, wider than the added entry's stop the driver hands over to a fresh build, and no added entry records that widening.
    fix: "Leave the code in place and defer this finding to the run's parent. The parent either records the wider classification, every failed halt on a newer board stencil, with lym loom decision add, or records that the kind must be narrowed. The fixer does not change halt.go for this finding."
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: "The change description's second design decision claims that when the on-disk binary is no newer, the driver hands the run back; for a failed producer stop the driver's rule table sends it to the tackler instead."
    fix: "In _lym/landing/summary.md, under Design decisions, second bullet, replace \"the driver hands the run back instead of over\" with \"the driver does not hand the run over, and its tackler settles the stop\"."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomengine/anomaly_test.go#TestClassifyHalt
    summary: No TestClassifyHalt row sets buildSkew with a non-failed state, so removing the StateFailed guard on the new build-skew case passes the whole table.
    fix: "Add a TestClassifyHalt row named NotFailedBuildSkew_NoKind with state shedengine.StateRunning, buildSkew true, wantKind \"\" and wantOK false."
---

### [LOW] Build-skew reason misstates the driver's action at three sites

**Location:** `contracts/stencils/friction/friction-template-reflection.md:48`, `internal/loomcli/doc.go:13`, `internal/loomengine/anomaly.go:31`

**Issue:** The stencil and the `loomcli` package doc both say the driver hands a build-skew run to a fresh driver of the newer build.
That holds only on the rule table's hand-over row, `binaryEligible(stop) && facts.BuildChanged && !facts.HandedOver` (`internal/sheddriver/driverules.go:264`).
When the on-disk build is unchanged, or its one hand-over is already spent, a failed producer stop has `stop.Kind == shedverbs.KindProducer` (`internal/shedverbs/step.go:422`).
That stop falls to `isTacklerKind(stop.Kind) && !stop.OK`, so the tackler decides (`driverules.go:275`, `isTacklerKind` at `driverules.go:305`).
The `AnomalyBuildSkew` doc's alternative, "or back to its parent to update the binary", is not a rule the table has either.
The affected-set lens proposed copying the `anomaly.go` wording into the other two sites, but that wording is wrong as well, so the fix rewords all three.
The drop rule itself stays correct, and every earlier drop and file rule in the stencil is intact.

### [LOW] Build-skew classification is wider than the added entry, without a recorded decision

**Location:** `internal/loomcli/halt.go#failedHalt`

**Issue:** The added entry, "Narrow to the false hard-failure report", covers "a stop the driver hands over to a fresh build".
`failedHalt` sets `BuildSkew: errors.As(err, &skew)` from the error alone (`halt.go:116,123`), before the driver decides.
A skew halt that the driver does not hand over is therefore also `build-skew`, and the reflection drops it.
The change description gives the reason for this choice: at note time the driver's decision is not yet known.
It records the choice as the writer's own design decision, not as a parent correction, and no added entry records it.
Whether those stops should be dropped too is the parent's call, so the finding is deferred to the parent and the code stays as it is.

### [LOW] Change description claims a hand-back the driver does not make

**Location:** `_lym/landing/summary.md`

**Issue:** The second design decision reads: "When the on-disk binary is no newer than the board, the driver hands the run back instead of over."
The diff does not support this claim.
For a failed producer stop with no build change, `decideDrive` reaches the tackler row (`internal/sheddriver/driverules.go:275`), not a hand-back row.

### [LOW] No row pins the failed-state guard on the build-skew case

**Location:** `internal/loomengine/anomaly_test.go#TestClassifyHalt`

**Issue:** The new case is `state == shedengine.StateFailed && buildSkew` (`internal/loomengine/anomaly.go:107`).
The added rows set `buildSkew` only together with `StateFailed`.
The doc comment says any state other than blocked or failed reports no kind.
If a mutant drops the state guard, a running state with `buildSkew` returns `AnomalyBuildSkew`, and no row fails.

## Rejected

- Nothing beyond the findings above. The correctness lens traced the real path: `stencilstore.Read` passes through `ReadSet`, `fillInstruction`, `composeReviewPrompt`, the `%w` wrap at `burlerreview.go:469`, `shedengine`'s `callErr` persist and return, `MarkTransient` with `Unwrap`, and `AfterStep`. `errors.As` sees the error on that path, and `st.Error == err.Error()` holds. No defect.
- The `Root-Cause:` trailer names the classification gap that the diff removes. The commit message makes no agreement or improvement claim that needs figures.
- Merge danger, `two-way` with blast radius `friction`: the new `anomaly:` value is parsed as an opaque string by `haltnote` and `reportengine` (`internal/reportengine/stops.go:150`). No closed kind set exists for another binary to break on, and the diff deletes no data and pushes nothing outward.
