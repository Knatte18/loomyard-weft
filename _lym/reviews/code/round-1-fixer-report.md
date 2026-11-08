# Fixer-report, code round 1

## Changes

Both changes are in `internal/hubgeom/hubgeom_test.go`, in `TestReedGeometry_ButtonContext`, committed as `5a57088d` and amended by the follow-up commit below.

- F1 fixed as proposed: row `BoardNonProjectWithBeamFlagReadsTask` uses `boardWith("child-task", "task", true)` and expects `ButtonPlaceTask`.
- F2 fixed as proposed: `boardWith` takes a slug parameter, the existing rows pass `child-task`, and row `BoardBeamOfAnotherSlugReadsTask` holds a beam-flagged project under `other-beam` and expects `ButtonPlaceTask`.
- Self-check LOW (fix-proposal): a comment above the F1 row says only a hand-edited board holds a beam flag on a non-project entry.

`lym gate test ./internal/hubgeom` passed (tier0, exit 0) after the rows were added.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: the F1 row pins a board state no write path can produce. Disposition: fix as proposed by the lens, a comment on the row; the row stays, as it is the only way to pin the kind operand.
- fix-proposal on F2: no findings.

## Disputed

None.

## Deferred

None.
