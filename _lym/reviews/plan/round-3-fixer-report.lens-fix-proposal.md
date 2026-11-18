```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F17 proposal that the fixer carried out in card 14.

**[MEDIUM]** Card 14's record-write paragraph, the new text: "A stop-ended relaunch writes nothing to the record, which stays stopped, and returns a Result whose Outcome is `OutcomeDied`, the shape `Wait`'s liveness read gives a strand a stop removed: on the `Wait` path beside nil". The load-turn sentence makes the same claim for the start path.

Summary: the proposal says the stop-ended relaunch takes the shape `Wait`'s liveness read gives, but it skips `finalize`, and that shape comes from `finalize`. The concrete break is a handle `Run.Stop` during a `Wait`-path relaunch, which ends the run with no session end reported.

Evidence:
- `Wait` reads a strand a stop removed as died (`internal/shuttleengine/doc.go:410-411`), then finalizes it.
- `finalize` (`internal/shuttleengine/wait.go:1978-2048`) does four things the new text drops:
  - It builds a full Result with SessionID, StrandGUID, RunDir, Engine, StartedAt and EndedAt. The fixed card returns a bare Outcome.
  - It runs `stampRunEnd`, which the code comment says lets a reader with no running run still see this one has just ended.
  - It records `stopOutcome` under `stopMarked`.
  - It calls `reportSessionEnd`.
- `Run.Stop` (`internal/shuttleengine/stop.go:43-57`) sets `stopMarked` and writes the record stopped, but it reports no session end. Only `stopRecorded`, the `StopStrand` form, calls `reportStoppedSession` (`stop.go:98`).
- Suppose a holder calls `run.Stop()` between the mark and step 4. The relaunch finds the record terminal, writes nothing and returns `Result{Outcome: OutcomeDied}`. Neither the stop nor the run reports a session end, and no run-end stamp is written. That breaks the card's own row "across a relaunch the run reports one session end".
- The new `StopStrand` row and the before-the-mark row only cover the guid-form stop, so no row catches this.

A sound version would separate the two stop forms. A handle stop, which has `stopMarked` set, could end through `finalize(OutcomeDied)`, which already records stopped through `stopOutcome`. A `StopStrand` stop, which already reported its session, could build the Result's identifying fields without a second report and still stamp the run end. The card could also simply stop calling the bare Result "the shape `Wait`'s liveness read gives".

**[LOW]** The new start-path row: "driven through `start`, the seam `Start` calls ... the result's outcome is died".

Summary: none of this card's other `Start`-driven rows go through `start`, and the shape `start` returns is only half pinned. `start` hands the result back only beside a nil run (`internal/shuttleengine/run.go:630-631`), and `Start` drops it (`run.go:540`). The row asserts `Outcome` but not the bare Result's missing fields, so it would pass whether the relaunch ran `finalize` or not. That leaves the MEDIUM above unpinned on the start path too.

Tool calls:
- grep for `reportSessionEnd`, `finalize` and `markedStopped` in `internal/shuttleengine`: my context did not show where session ends are reported.
- read of `internal/shuttleengine/wait.go` lines 1978-2057 (`finalize`): my context did not show what `finalize` writes and reports.
