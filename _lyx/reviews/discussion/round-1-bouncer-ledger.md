---
round: 1
ledger:
  - key: marker-placement-between-d1-d2
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: dropped-lyx-guard-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d4-bound-overstates-coverage
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: acceptance-criterion-2-contradicts-d1-keeps
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: merge-review-rubric-unexempted-coverage-item
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: class-first-card-note-leaves-fate-open
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all open.
The gating one is `d4-bound-overstates-coverage`: D4 credits Plan-Review with a guard it does not hold and ignores landings that pass no rubric.
`dropped-lyx-guard-unbounded` also changes a Decision.
The other four are local corrections whose text the review fixes.
