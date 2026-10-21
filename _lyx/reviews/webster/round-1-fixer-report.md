## Changes

- F1 (BLOCKING), fixed as proposed: `PATTERN.md` restores the `PATTERN-push-both-sides` and `PATTERN-leaf-packages` overview lines to their merge-base text.
  Only the declared `PATTERN-friction-capture` change remains in the `PATTERN.md` diff.
  Commit `PATTERN: fix F1`.
- F2 (LOW), fixed as proposed: `resettle` in `internal/shedadapters/bouncer.go` returns a `readCirclingDecision` error, wrapped with `%w` and naming the Bouncer and the round, before choosing the closing.
  The new `TestBouncer_Reentry` row `MalformedCirclingDecisionOnASettledRoundFailsTheReSettle` writes a malformed decision file after a failed-commit settle and expects the error and no second carry-over.
  Commit `shedadapters: fix F2`.
- F3 (NIT), fixed as proposed: the bare-pause branch of `pauseCmd` in `internal/shedverbs/pause.go` assigns `PauseRequestedBy` only when `--by` is non-empty or no pause was requested yet.
  `TestPauseCmd_SetsPauseRequested` gains an assertion that a repeat bare pause keeps the earlier `operator` label.
  Commit `shedverbs: fix F3`.

- Gate repair: restoring the two F1 lines grew the `PATTERN.md` overview to 15820 bytes, past the 15564-byte headroom line.
  The `PATTERN-friction-capture` overview line is shortened to "Go writes a note, reflected at the run's end, archived once covered.", which brings the overview to 15746 bytes.
  The rule's detail already lives in `pattern/PATTERN-friction-capture.md`, so no background file changed.
  Commit `PATTERN: shorten the friction-capture overview line`.

`lyx gate test ./internal/...` is green after the repair.
The full `integration` run was not made, since the round never pushes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: one LOW note on `internal/shedadapters/bouncer.go:1004` (`standsAsRecorded`), which discards the `readCirclingDecision` error.
  Not a finding under the rubric: the review named only `resettle`, and a malformed file reports `exists=true`, so the site still treats the round as recorded and the error surfaces at the next settle.
  No change made.

## Disputed

None.

## Deferred

None.
