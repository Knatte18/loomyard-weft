---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: "lens:code-correctness"
    location: "internal/driftengine/check.go:482"
    summary: "the Terminal drift-gate findings drop the records and seal faults that NewDriftGate's doc and doc.go say they name"
    fix: "In `unchangeableFindings` in `internal/driftengine/check.go`, also render `r.RecordsFault` and `r.SealFault` as `records fault` and `seal fault` lines, as `Findings()` renders them, ahead of the unchangeable-cell text. Add a row to `TestDriftGateUnchangeableCells` where a records or seal fault coincides with unchangeable cells and the findings name the fault."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: "lens:code-tests"
    location: "internal/driftengine/verify.go:879"
    summary: "the diff-command rule that a judge's ruling on a marked case counts as matches, and a stored one stops nothing, has no test"
    fix: "Add a row to the Drift-Verify tests in `internal/driftengine/verify_test.go` under `diff: command`. Mark a case under `metadata.files`, store a non-matching ruling for it at the code head, and assert `storedCaseRulings` raises no Stuck. Add a second row where the judge rules `does not match` on the marked case, and assert the ruling is counted as matching and not stored by `storeCaseRulings`."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: "lens:code-tests"
    location: "internal/driftengine/check.go:190"
    summary: "Check dropping the cells of metadata marks that arrive by lym drift amend --entries, the Terminal stop's own way forward, has no test"
    fix: "Add a row to `TestCheck` in `internal/driftengine/check_test.go`. Seed a scope record with `Amendments` holding one `entries` amendment whose text is `expectsMetadata`, set `f.current` to `metadataCurrent`, and assert the report has no cell, carries the marks in `report.Metadata`, and the gate passes."
  - id: F4
    severity: LOW
    class: consistency
    origin: "lens:code-affected-set"
    location: "internal/driftengine/check.go:396"
    summary: "the Terminal stop's way forward ends in lym loom resume, and the writer exit appends a second way forward saying that resume starts every part over"
    fix: "In `unchangeableWayForward` in `internal/driftengine/check.go`, drop the trailing `then lym loom resume` clause and name only the amend, since the writer exit in `internal/shedadapters/multillm.go` appends its own resume way forward to every Stuck reason. Alternatively word the clause so it does not contradict that appended text."
  - id: F5
    severity: LOW
    class: design
    origin: "lens:code-correctness"
    location: "internal/driftengine/capture.go:208"
    summary: "backUpBaseline can leave the baseline moved and its metadata behind when baseline.N.meta already exists, and never clears a stale baseline.meta beside an empty baseline"
    fix: "In `backUpBaseline` in `internal/driftengine/capture.go`, remove or move aside a stale `metaDirFor(baselineDir)` when the baseline directory is empty, and pick the backup number so neither `baseline.N` nor `baseline.N.meta` exists. Do both before the first rename, so a failed metadata move cannot follow a moved baseline."
  - id: F6
    severity: LOW
    class: scope
    origin: "lens:code-tests"
    location: "internal/driftengine/check.go:245"
    summary: "second-capture and commit-rule branches, namesCommit boundaries, meta_dir in compare and the Drift.Head refusal have no row"
    fix: "Add rows to `TestDriftGateUnchangeableCells` in `internal/driftengine/check_test.go` for a second capture that does not pass (nothing marked, gate fails without Terminal), a cell absent from the second capture, a cell of another `Kind` in it, and `baseHead == head`. Add a `namesCommit` row set around the seven-digit minimum and a non-hex separator. Add a `{meta_dir}` render row to the `RunCompare` tests in `internal/driftengine/compare_test.go`. Add a refusal row for a missing `Drift.Head` to the `shedrecipe` drift-seam test."
---

## Findings

### [MEDIUM] Terminal findings drop the records and seal faults

**Location:** `internal/driftengine/check.go:482`
**Issue:** `unchangeableFindings` renders only the unchangeable cells and the way forward. The `NewDriftGate` doc comment and `doc.go` say the failure's findings also name a records or seal fault found beside it. That holds on the non-terminal path through `Findings()`. On the Terminal path a records mismatch or a failed seal check is hidden from the parent's one stop.

### [MEDIUM] No test for the diff-command ruling change on marked cases

**Location:** `internal/driftengine/verify.go:879`
**Issue:** `storedCaseRulings` and `storeCaseRulings` now treat a ruling on a marked case as `matches`. The landing summary claims it, and the board entry asks that no later check fails on a mark under either diff mode. No changed row in `verify_test.go` pins it.

### [MEDIUM] No test that amended metadata marks drop cells in Check

**Location:** `internal/driftengine/check.go:190`
**Issue:** The Terminal stop's way forward is `lym drift amend --entries` with `metadata` marks. `Check` reads them through `sealedAndAmended` and `Metadata.union`. Only `Merge` is tested; no `TestCheck` row seeds an amendment holding marks and expects a cell-free report.

### [LOW] Two way forwards at the Terminal stop

**Location:** `internal/driftengine/check.go:396`
**Issue:** `writerExit` appends `writerResumeWayForward` to every Stuck reason, so the parent's one stop names `then lym loom resume` and then says a resume starts every part over. The entry wants the stop to be one clear question.

### [LOW] Half-moved baseline when the backup's metadata slot is taken

**Location:** `internal/driftengine/capture.go:208`
**Issue:** `nextFree` checks `baseline.N` and never `baseline.N.meta`. A leftover non-empty `baseline.N.meta` makes the metadata rename fail after the baseline has moved. An empty baseline directory also leaves an old `baseline.meta` beside the next capture. Neither affects what drift compares.

### [LOW] Untested branches of the gate and its helpers

**Location:** `internal/driftengine/check.go:245`
**Issue:** `TestDriftGateUnchangeableCells` has three rows. A failed second capture, a cell missing from the second capture, a changed `Kind`, `baseHead == head`, the `namesCommit` boundaries, `{meta_dir}` in the compare command and the new `Drift.Head` refusal each have no row. One finding covers the set, since they share one gap.

## Rejected

- Correctness lens finding 3, that a flaky result is called capture metadata in the Terminal text: the board entry names cells that differ between two captures of the same code as in scope, and the text states the rule that marked them.
- Affected-set lens remark that the driver's own stop after a Terminal gate result was not traced: the shed driver holds no drift-specific rule, and the stop is the generic one the board entry's Seen section already covers, so no second message is shown by the code read.

## Holistic check against the board entry

- Lym writes no `_source.txt`, and the diff adds nothing the entry does not license.
- The metadata directory is a `.meta` sibling of `{baseline_dir}`, never listed, compared or shown; the template and package doc say provenance goes there; the stencil has the expect agent mark metadata under both diff modes; the scope record carries the marks from the seal on.
- Mixed failures still re-prompt, and only an all-unchangeable failure ends the gate at once.
- `## Regression check` names the baseline commit and the marks.
- The Models hub harness change is named in the landing summary's follow-up section.
- No `Pattern-Departure:` trailer is needed and no PATTERN entry is declared changed.
