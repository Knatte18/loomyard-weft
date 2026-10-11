# Fixer report — discussion, round 2

All premises were checked against the code: `Goto` in `internal/shedengine/goto.go` clears `PauseRequested`, `PauseBefore` and `PauseAfter`, `resumeFromRunState` in `internal/loomcli/resume.go` writes no status, and `stepLocked` in `internal/shedengine/run.go` short-circuits only `StateDone`.

## Changes

- F1 (BLOCKING), fixed as proposed, with one addition.
  D3 drops `lyx loom goto` from the arbiter's verbs; a `resume` verdict may name a goto target instead, and D3's bound now says the arbiter names an earlier row that the re-launch moves the run back to.
  D1 records `pause_before` and `pause_after` in the attempt at spawn, and its applied mark is written before the verdict's goto.
  D6's re-launch now consumes the attempt under the status lock.
  It checks the stop key, an unset `pause_requested` and the recorded pause conditions, then writes the mark, performs the goto and steps the run.
  Any other state is refused before the goto.
  D6's `blocked` precondition now reads "the verdict names a goto target the re-launch admits", and an error-stop `resume` may name one.
  D2's bound covers the moved goto, and the stencil's goto sentence in D6 is updated.
  Open risks and the auto-mode assumptions are reworded to match, and a new acceptance criterion covers an orch `pause` (plain, `--before` or `--after`) during a live arbiter whose verdict names a goto target.
  The addition: the re-launch's goto keeps `pause_before` and `pause_after` rather than clearing them as `lyx loom goto` does.
  Without this, a pause condition set before the stop, which the check admits because it equals the recorded value, would still be cleared by the goto, and a human's pause would be removed.
- F2 (MEDIUM), fixed as proposed: the un-pause clause in D6 is replaced with the sentence saying a re-launch steps an `awaiting` or `paused` run as it stands.
  The support-log ledger entry is resolved by the code, and the Notes bullet no longer asks the Plan to check it.
- F3 (LOW), fixed as proposed: D7 hands on an applied attempt with an unchanged stop key whether the stop recurred or the re-launch never stepped.
  Its reason reads "an applied resume left the stop key unchanged", and it costs one parent wake and no second re-launch.
  An acceptance criterion covers the never-stepped case.
- F4 (NIT), fixed as proposed: D2's list of Go inputs names the Publish row's verify-failure record.
- support-log.md gains a Round 2 entry under Review rounds.

## Disputed

None.

## Deferred

None.
