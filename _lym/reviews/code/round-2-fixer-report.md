# Round 2 fixer-report

## Changes

- F1 (LOW): fixed as proposed, with one detail of my own.
  In `pollEventsTick`, `expiredTurnEnd` now runs first and `queueShellNudge` only when it returns an empty outcome (`internal/shuttleengine/wait.go`).
  The held result is discarded with `_`, since the gated branch never returns one, so no local shadows the `heldTurnEnd` type.
  Test: a new `TestWait_GatedShellNudge` row seeds `WAIT` then `START`, ages the clock past the bound, appends a second waiting turn end over the same shell, and fails the gate once.
  It asserts no nudge is typed.
  I confirmed the row fails with the old order restored and passes with the fix.
  Commits: `86d59f4f`, then `shuttleengine: fix self-check twins of F1`.
- F2 (NIT): fixed as proposed, "With both present" became "With every output file present" in `internal/websterengine/doc.go`. Commit `73110a24`.

The gate `lym gate test ./internal/shuttleengine` passes.

## Self-check

- fix-code-correctness, LOW, shadowed `heldTurnEnd`: fixed as proposed (discarded with `_`).
- fix-code-correctness, LOW, test doc comment stale for the new row: fixed as proposed.
- fix-consistency, LOW, `pollEventsTick` doc still says it queues the nudge before `expiredTurnEnd`: fixed as proposed.
- fix-consistency, LOW, test doc comment contradicts the new row: fixed as proposed (duplicate of the fix-code-correctness finding).
- fix-consistency, LOW, `ageShellThen` comment misdescribes `shellAger`: fixed as proposed.
- fix-proposal, MEDIUM, the new row passes with the defect in place because the gate passes and the nudge is never sent: fixed as proposed.
  The row now fails the gate once and counts nudges only; the red run is confirmed.
- fix-proposal, LOW, shadowed `heldTurnEnd`: fixed as proposed (duplicate).
- fix-affected-set: no findings. It noted that `runlevel.go:1` and `runlevel.go:768` also say "both output files" for the Master spawn; those lie outside the review's finding and paragraph, so I left them.

## Disputed

None.

## Deferred

None.
