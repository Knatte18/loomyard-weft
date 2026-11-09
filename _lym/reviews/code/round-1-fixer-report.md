# Fixer-report, code round 1

## Changes

- F1 (fixed as proposed, first of the two options): `pattern/PATTERN-test-speed.md` under "Time is injectable" gained an exception bullet.
  A cost budget no outcome can stand in for may compare a measured time to a bound far above what load adds to the work it measures.
  The rule and `startCostBound` now agree.
  The first draft tied the exception to a `//testtiming:keep` marker; the self-check showed the marker means something else, so the final text drops that clause (see Self-check).
- F2 (fixed): the `//testtiming:keep` marker on `TestBattenIntegration_Rows` no longer cites "100 ms" or D2.
  It says the step pins the start-cost bound on a Command row beyond its status commit and its process.

Commits: `pattern: fix F1`, `battencli: fix F2`, then one follow-up each for the self-check changes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: the exception tied measured-time licence to `//testtiming:keep`, which the PATTERN defines as the redundancy-report marker. Fixed: clause dropped.
- fix-consistency, LOW: the marker names `startCostBound`, defined far below. Fixed: marker reworded without the "under startCostBound" claim; it states what is pinned.
- fix-proposal, MEDIUM: same as the consistency MEDIUM (merged). Fixed as above.
- fix-proposal, LOW: the marker said a row waiting seconds "fails", stronger than a 2 s bound. Fixed: claim removed from the marker.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
