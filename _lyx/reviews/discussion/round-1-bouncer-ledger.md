---
round: 1
ledger:
  - key: amends-plan-commit-failure-land-together
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d9-trimpath-panic-paths-dropped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-no-per-card-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-amendment-target-carrier
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d7-d8-reopen-and-widening-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-removal-time-comparison
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-answer-key-clauses-disagree
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d2-escape-bound-understated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d8-offer-before-plan-approval
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: d3-resign-arrow-under-extends
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d9-overlap-rule-misattributed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-no-parent-headroom-disagree
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-removes-vs-fenced-order
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d1-way-forward-missing-amends-plan
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-opaque-identity-bare-locations
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stencil-decisions-no-acceptance-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-justification-and-duplicate-risk
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger exists.
Seventeen findings map to the keys above, F1 to F17 in review order.
F1 is the only BLOCKING one: the `--amends-plan` verb's land-together guarantee fails on a commit failure.
The design-class MEDIUMs (F2, F3, F5, F6, F8) concern D9 trace placement, D7 and D8 bounds, D7 removal timing and D2's escape bound.
Findings whose fix changes a decision, a contract or an interface are marked `design-wrong`, so a fresh reviewer sees the new text; local corrections are `mechanical`.
