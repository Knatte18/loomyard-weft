---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/boardengine/doc.go:315
    summary: The Sketch doc and the change description say the entry's dates stay, but the write restamps Revised, and no test asserts dates
    fix: In internal/boardengine/doc.go, the "# Sketch" paragraph that lists what stays ("its title, brief, body, project, issues, dates and discussed flag stay"), change "dates" to "Created date" and add a clause that Revised is stamped by the write like any scope change. Make the same correction in the "Its title, brief, body, project, issues, dates and discussed flag" bullet of _lym/landing/summary.md. In internal/boardengine/sketch_test.go, in the HeldTaskBecomesASketch subtest, assert that Created equals its value before MakeSketch, and that Revised is not before it.
  - id: F2
    severity: LOW
    class: scope
    location: internal/boardengine/sketch_test.go:134
    summary: The exemption test has no row for a Sketch note that keeps its run status, the only case that exercises the now.Status == nil clause of endsAsSketch
    fix: In TestRunLock_SketchExemptionIsTheWholeNetChange, add a row "SketchThatKeepsItsRunStatusIsRefused" whose after is sketch(func(task *boardengine.Task) { task.Status = &running }) with wantErr true. Without it, dropping the now.Status == nil clause in endsAsSketch (internal/boardengine/sketch.go:67) leaves every test green and lets a held note with a run status through the run lock.
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:62
    summary: The change description says no test drives a motif finish on an entry something depends on, but the diff adds that test
    fix: In _lym/landing/summary.md under "To check by hand", delete the bullet "The fallback to draft when MakeSketch is refused is covered only by reading the code. No test drives a motif finish on an entry that something depends on." TestWiredCommitStatus_FinishedRunEntry in internal/loomcli/commitstatus_integration_test.go reaches it with the RefusedSketchLeavesTheEntryDraft row.
---

### [LOW] The Sketch doc and the change description say the entry's dates stay, but the write restamps Revised

**Location:** `internal/boardengine/doc.go:315`, `_lym/landing/summary.md`
**Issue:** `doc.go`'s Sketch section and the summary both say the entry's dates stay.
`stampDates` in `internal/boardengine/dates.go` sets Revised to now whenever `scopeChanged(before, after)` holds and the write did not set Revised itself.
`MakeSketch` changes Kind, Labels and Recipe, so `scopeChanged` holds and Revised moves to the time of the write.
Created is the only date that stays.
`sketch_test.go` asserts project, brief, body and issues as kept fields but never the dates, so nothing notices either reading.

### [LOW] The exemption test has no row for a Sketch note that keeps its run status

**Location:** `internal/boardengine/sketch_test.go:134`
**Issue:** `endsAsSketch` requires `now.Status == nil`, and `scopeChanged` does not compare Status, so that clause is the only thing refusing a held task that becomes a Sketch note while keeping its run status.
The four rows of `TestRunLock_SketchExemptionIsTheWholeNetChange` are the plain Sketch, a title change, a body change and a note without the sketch label.
None of them sets Status on the after entry, so a mutation that drops the clause passes the whole suite.

### [LOW] The change description says no test drives a motif finish on an entry something depends on

**Location:** `_lym/landing/summary.md:62`
**Issue:** The "To check by hand" section says the fallback to draft is covered only by reading the code.
The diff adds the `RefusedSketchLeavesTheEntryDraft` row to `TestWiredCommitStatus_FinishedRunEntry`.
That row creates an entry that lists the run's entry in `depends_on` and asserts the run's entry ends as a task in status `draft`.
The claim does not hold.

## Rejected

- Focus 1, `checkRunLocks` and a Sketch conversion with another scope change: not a defect.
  `endsAsSketch` requires `before.Kind == task`, a status-less note carrying the sketch label, and `!scopeChanged(sketchOf(before), now)`.
  `sketchOf` keeps title, brief, body, short name, project, beam, order, issues, priority, front, on_land and the module labels, and `scopeChanged` compares each of them.
  A change to any one of them, or an absent slug, is refused with `ErrRunLocked`.
  A note without the sketch label differs in Labels from `sketchOf`'s result and is refused.
  The title and body rows and the missing-label row are tested; the other fields rely on the shared `scopeChanged`.
- Focus 2, `hubStartGuard`'s Unreleased closure: not a defect.
  The closure returns the status for `StatusDraft` and `LegacyStatusDiscussed`, so `lym batten run` refuses a depended-on task still carrying `discussed`.
  The field comment, the `refuse` doc comment, the refusal text and `releaseCommandText` all name draft and discussed.
  `TestStartGuard_Refuse` pins the discussed refusal through the seam.
- Focus 3, `sketchFinish`: not a defect.
  The fall back to draft happens only when `errors.Is(err, boardengine.ErrSketchRefused)`.
  `MakeSketch` wraps only the `validateWrite` error with that sentinel, and every other error is returned for the seam's next finished transition.
  `TestWiredCommitStatus_FinishedRunEntry`'s `RefusedSketchLeavesTheEntryDraft` row reaches the refused branch with an entry that lists the run's entry in `depends_on`.
- Focus 4, coverage of the held-task conversion: the cleared fields (isolated, profile, pull_request, notify_on_land), the kept fields (project, brief, body, issues), a depended-on discussed task, a replayed Sketch write on a held entry and `MakeSketch` on an existing Sketch are all covered.
  The dates and the retained-status case are the gaps, filed as F1 and F2.
  The exemption boundary is pinned through `CheckRunLocks`, and the whole-write path through `MakeSketch`.
