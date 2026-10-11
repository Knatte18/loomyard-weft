# Fixer report — discussion, round 3

## Changes

- **F1** (MEDIUM), fixed as proposed.
  Premise confirmed: `internal/shedengine/goto.go` clears `PauseRequested`, `PauseBefore` and `PauseAfter` in its `state.UpdateJSON` persist, `internal/shedverbs/pause.go` sets `PauseRequested` under the status lock alone, and the step's persist in `internal/shedengine/run.go` takes the status lock again.
  D6's resume bullet now puts the state check, the applied mark and the goto in one status-file write under one hold of the status lock, and steps the run after that hold is released.
  The re-launch's goto now keeps `pause_requested`, `pause_before` and `pause_after`.
  Acceptance criteria gain: a plain `pause` landing after the verdict and before the step survives the re-launch and pauses the run.
- **F2** (LOW), fixed as proposed.
  D6's resume bullet now takes the run lock before consuming the attempt, so a busy refusal leaves the attempt unapplied.
  D7's examples now name a driver dead after the mark and a step that failed before its first status write.
  The applied-resume acceptance criterion now names a driver that died between the applied mark and the step.
  The round-2 "Added after Discussion" entry's rationale still mentions a busy refusal; it is the parent's dated record of that round and was left as written.
- **F3** (LOW), fixed as proposed.
  The support log's recommend-only rejection now names only `circling accept` as unchecked and states that the re-launch's goto is checked in Go against the attempt's recorded state (D6).

## Disputed

None.

## Deferred

None.
