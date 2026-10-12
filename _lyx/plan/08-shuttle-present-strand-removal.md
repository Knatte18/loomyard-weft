# Card 8 — shuttle removes a tracked own-window strand whose pane is dead, and keeps a died run's capture

**Create:**
- `plan:internal/shuttleengine#strandRemovable` -> `func strandRemovable(strand reedengine.StrandStatus) bool`
- `plan:internal/shuttleengine#removeStrandIfPresent` -> `func removeStrandIfPresent(reed ReedOps, guid string) error`
- `plan:internal/shuttleengine#diedCaptureFileName` -> `const diedCaptureFileName = "died-capture.txt"`
- `plan:internal/shuttleengine#Run.removeDiedOwnWindowStrand` -> `func (run *Run) removeDiedOwnWindowStrand()`
- `plan:internal/shuttleengine#cleanupDiedCaptureKept` -> `const cleanupDiedCaptureKept cleanupExpectation = 3`

**Delete:**
- `internal/shuttleengine#removeStrandIfLive`

**Edit:**
- `internal/shuttleengine#Run.Stop`
- `internal/shuttleengine#Runner.StopStrand`
- `internal/shuttleengine#Runner.stopRecorded`
- `internal/shuttleengine#Runner.StopStepRuns`
- `internal/shuttleengine#Runner.removeSupersededStrands`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#auditedNegativeVerdictReturns`
- `internal/shuttleengine#TestStop_RecordsTheStopBeforeRemovingTheStrand`
- `internal/shuttleengine#TestRunner_StopStepRuns_StopsOnlyTheKilledStepsLiveRuns`
- `internal/shuttleengine#TestAttach_RemovesSupersededStrands`
- `internal/shuttleengine#TestRun_Wait_Classification`

**Uses:**
- `internal/shuttleengine#strandStatusByGUID`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#newFixture`

**Intent:**
D5's shuttle half: a dead agent's own window goes with the run that owns it, through the stop verb, the step stop, supersession on respawn, a seat table's stop and `Wait`'s died exit, while window 0 strands keep today's live-only removal.
It reads the `Present` and `OwnWindow` fields card 3 added to `reedengine.StrandStatus`.

New in `internal/shuttleengine/stop.go`:

- `strandRemovable(strand)` is true for a live strand, and for an own-window strand whose pane is present but dead (`strand.OwnWindow && strand.Present`); a window 0 strand with a dead pane is not removable, as today, since reconcile kills its pane at the next reed op.
- `removeStrandIfPresent(reed, guid)` replaces `removeStrandIfLive`: one `Status` probe, nothing done for an untracked strand or one `strandRemovable` rejects, otherwise a Warn naming the strand (`shuttle: stopping a strand`, with `live`) and one `RemoveStrand(guid, false)`.
  A probe that could not answer still returns an error rather than guessing.
  It removes only the strand the caller names, which is always a run record's strand or the guid `StopStrand` was given, never an untracked pane.
- `removeStrandIfLive` is deleted; its three callers, `Run.Stop`, `Runner.StopStrand` and `Runner.stopRecorded`, call `removeStrandIfPresent`, and their doc comments say they remove a live strand or a dead own-window one.

`Runner.StopStepRuns` skips a run only when its strand is untracked or `strandRemovable` rejects it, so a step stop of a run whose own-window agent died ends that window; its doc comment says so.
`Runner.removeSupersededStrands` does the same, so a respawn superseding a dead own-window run removes the dead strand first.
A seat table stops every advisor through `Run.Stop` before its step returns (`seatengine`'s `join`), so a dead advisor's strand goes too and a shared seat window ends with its table.

`Run.removeDiedOwnWindowStrand()`, in `wait.go`, runs from `Run.Wait` right after each of its `finalize` calls whose result is `OutcomeDied` (the events branch's finalize and the liveness exit's), so every died exit of the wait loop takes it:

- One `Status`; it acts only when the run's own strand is tracked, own-window and present.
- It captures the pane through `CapturePane` and writes the capture to `<runDir>/died-capture.txt`; a failed capture or write is a Warn and the removal still runs.
- It removes the strand with `RemoveStrand(guid, false)`; a failure is a Warn naming the strand and `lyx reed remove` as the way forward.
- It logs `shuttle: removed a died run's own-window strand` at Info with the run dir.
- A timed-out run never reaches it, so its strand stays, since its agent may still be live.
- The startup teardown (`abandonStartup`) keeps its own removal and capture, so a run that died before it was ready is removed once.

`internal/shuttleengine/doc.go`'s stop-verb paragraph says the stop removes a live strand or a dead own-window one; the died paragraph says a died run whose strand is own-window keeps its last capture in `died-capture.txt` and loses its strand.

Tests:

- `TestStop_RecordsTheStopBeforeRemovingTheStrand`, which runs every row through both the handle form (`Run.Stop`, the form a seat table's `join` uses) and the guid form, gains rows `dead_own_window_strand_is_recorded_and_removed` (`{GUID: "strand-1", PaneID: "%1", OwnWindow: true, Present: true}`: recorded stopped, removed) and `own_window_strand_with_no_present_pane_is_recorded_and_not_removed` (`OwnWindow: true`, not present: recorded stopped, not removed); the existing `dead_strand_is_recorded_and_not_removed` row keeps the window 0 behavior.
- `TestRunner_StopStepRuns_StopsOnlyTheKilledStepsLiveRuns` adds `{GUID: "guid-dead-own", PaneID: "%6", OwnWindow: true, Present: true}` and `{GUID: "guid-dead-window0", PaneID: "%7", Present: true}` to its strands slice and seeds a tied running run on each.
  Its assertions change to match: `report.Stopped` is `["name-tied", "name-dead-own"]` in run-directory order, the removals are `guid-tied` and `guid-dead-own`, and the outcome map adds `dead-own` stopped and `dead-window0` running.
- `auditedNegativeVerdictReturns`'s key `removeStrandIfLive [Errorf]` becomes `removeStrandIfPresent [Errorf]` with the same count of two (the failed probe and the failed removal), and the matching bullet of its doc comment names the new function, so `TestCompletionSignal_NegativeVerdictReturnSites` keeps passing on `stop.go`.
  `removeDiedOwnWindowStrand` returns nothing, so `wait.go` gains no audited return.
- `TestAttach_RemovesSupersededStrands` gains `dead_own_window_candidate_is_removed`: outcome `died`, strand `{GUID: "strand-1", PaneID: "%1", OwnWindow: true, Present: true}`, removed and recorded `died`; the existing `dead_pane_candidate_removes_nothing` row keeps the window 0 behavior.
- `TestRun_Wait_Classification` gains the expectation `cleanupDiedCaptureKept`, added to the `cleanupExpectation` const block in `wait_test.go` after `cleanupSkipped` (the strand removed, the run dir kept, and `died-capture.txt` holding the scripted capture) and a `capture string` row field fed to `reed.CaptureQueue`, and the row loop's `switch tt.wantCleanup` gains its case: `RemoveStrand(strand-1, false)` recorded, the run dir still present, and `<runDir>/died-capture.txt` equal to the row's capture.
  Rows (the existing `dead pane is died and keeps the strand` row already pins the window 0 died exit):
  - `a died own-window run keeps its capture and loses its strand`: status `{GUID: "strand-1", PaneID: "%0", OwnWindow: true, Present: true}`, started, capture `"last screen"`: `OutcomeDied`, `cleanupDiedCaptureKept`.
  - `a timed-out own-window run keeps its strand`: a live own-window strand past its deadline: `OutcomeTimeout`, `cleanupSkipped`.

**ImpactSummary:** Renames the stop helper (three callers in stop.go and the completion-signal audit key, all on this card) and widens three removal sites to dead own-window strands; window 0 strands, and every reed state that reports no own-window field, behave exactly as before.

**Commit:** 8: Shuttle removes dead own-window strands and keeps a died run's last capture
