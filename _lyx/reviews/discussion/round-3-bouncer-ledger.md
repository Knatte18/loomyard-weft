---
round: 3
ledger:
  - key: marker-placement-between-d1-d2
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dropped-lyx-guard-unbounded
    rounds: [1]
    status: resolved
  - key: d4-bound-overstates-coverage
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: acceptance-criterion-2-contradicts-d1-keeps
    rounds: [1, 2, 3]
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
  - key: kickstart-position-row-looser-than-criterion
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Three rounds: six findings, then three, then four, none BLOCKING.
Three round-1 keys stay resolved: the dropped-guard bound, the merge-review exemption and the class and first-card note.

`d4-bound-overstates-coverage` has been a MEDIUM `design` finding in every round.
Each round closes one route (round 2 the zero-budget runs) and finds another that skips the judged review (round 3 the auto-accept and hard-cap routes).
`marker-placement-between-d1-d2` also recurs, now as a `design` finding: D2 sends a marker-naming row away from a stencil no other test reads, so a marker can vanish unseen.
`acceptance-criterion-2-contradicts-d1-keeps` recurs as D2's section-scoped definition catching the Step 5 `specs_dir` row, which carries a prose path.
The new LOW key is the `kickstart_pack_inline` row and its criterion allowing moves the row would not catch.
