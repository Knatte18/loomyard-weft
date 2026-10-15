---
round: 2
ledger:
  - key: marker-placement-between-d1-d2
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: dropped-lyx-guard-unbounded
    rounds: [1]
    status: resolved
  - key: d4-bound-overstates-coverage
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: acceptance-criterion-2-contradicts-d1-keeps
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: merge-review-rubric-unexempted-coverage-item
    rounds: [1]
    status: resolved
  - key: class-first-card-note-leaves-fate-open
    rounds: [1]
    status: resolved
---

Round 1 raised six findings; round 2 raised three, all MEDIUM, none BLOCKING.
Three round-1 keys no longer appear: the dropped-guard bound, the merge-review exemption and the class and first-card note.

The gating one is `d4-bound-overstates-coverage`: round 1 found D4 crediting a guard it does not hold, and round 2 finds the bound still wrong for zero-budget loom and darn runs, which skip their review perch.
Its fix changes D4's bound, so it needs a fresh review.
`marker-placement-between-d1-d2` recurs as D2 dropping section-scoped marker rows that D1 keeps.
`acceptance-criterion-2-contradicts-d1-keeps` recurs as the kept kickstart rows still anchoring on prose and a heading the criterion does not name.
Both are local corrections.
