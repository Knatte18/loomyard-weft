---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

## Changes

- F1 (`50f8d92e9`): added `RoundWithNoTargetIdCountsAsChanged` to `TestBouncer_Stalemate` and `ARecordWithNoTargetIdsIsVerified` to `TestBouncer_FixVerification`.
  Both rows failed with the two empty-id guards temporarily removed, and pass with them restored.
- F2 (`dafc19855`): added the row `a target that matches no file leaves no target id` to `TestBurlerReviewProducer_Call_RecordsRoundSnapshot`.
- F3 (`7d2945fd2`): changed the `TargetSnapshot.Paths` comment as proposed.
  The same stale "worktree-relative" wording was also dropped from the comments on `roundChangesPath` and `RoundContext.HasChanges`.
- F4 (`_lym/landing/summary.md`, not committed): the blast radius is now `review`.

`lym gate test ./internal/shedadapters ./internal/burlerengine` passed at tier0: 1796 passed, 0 failed, 4 skipped.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW (the summary's Tests section lists none of the added rows): fixed, in `_lym/landing/summary.md`, not committed.
- fix-proposal: no findings.
- fix-affected-set: no findings.
  Its side note, that anchor-relative changed paths might not match an absolute finding location on a subpath-anchored hub, is no defect:
  `burlerGeometry` in `internal/hubgeom/hubgeom.go` sets `WorktreeRoot` to the anchor path.

## Disputed

## Deferred

## Rulings
