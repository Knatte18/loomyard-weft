# Fixer-report, darn round 1

## Changes

- F1 (MEDIUM), fixed as proposed: the merge-parent row in `internal/reportengine/landings_test.go` gains `pairOpening("open-merged", ...)` below its trailer commit and `pairOpening("open-other", ...)` above it; the expected exact-join mapping is unchanged.
- F2 (LOW), fixed differently: the proposal (an opening commit below the trailer in the squash row) leaves the walk returning at the trailer, so the floor is never read on the later page.
  The squash row got the opening commit anyway, and a new row puts the opening commit past the first page with no trailer above it, with the parent's trailer below it; the walk must stop at the opening commit and the tag maps through the time join.
- F3 (LOW), fixed as proposed: `TestAdd_OriginRecord`'s first subtest in `internal/fabricengine/origin_integration_test.go` asserts the weft branch tip subject equals `OriginRecordSubject(slug)` and the literal `fabric: record parent branch for <slug>`.

Gate: `lyx gate test --tier 1 ./internal/reportengine ./internal/fabricengine` green after F1 and F3; tier 0 of `./internal/reportengine` green after the F2 row.

## Self-check

- fix-code-correctness, LOW: the squash row's opening commit does not pin the floor on a later page. Fixed with my own change (the new row above).
- fix-proposal, LOW: same finding as above. Merged and fixed the same way.
- fix-affected-set: the same gap noted as the floor having no later-page row. Merged and fixed the same way.
- fix-consistency: no findings.

## Disputed

None.

## Deferred

None.
