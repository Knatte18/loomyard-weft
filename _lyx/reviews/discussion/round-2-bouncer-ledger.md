---
round: 2
ledger:
  - key: amends-plan-commit-failure-land-together
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d9-trimpath-panic-paths-dropped
    rounds: [1]
    status: resolved
  - key: d9-no-per-card-commit
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-amendment-target-carrier
    rounds: [1]
    status: resolved
  - key: d7-d8-reopen-and-widening-unbounded
    rounds: [1]
    status: resolved
  - key: d7-removal-time-comparison
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-answer-key-clauses-disagree
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d2-escape-bound-understated
    rounds: [1]
    status: resolved
  - key: d8-offer-before-plan-approval
    rounds: [1]
    status: resolved
  - key: d3-resign-arrow-under-extends
    rounds: [1]
    status: resolved
  - key: d9-overlap-rule-misattributed
    rounds: [1]
    status: resolved
  - key: d1-no-parent-headroom-disagree
    rounds: [1]
    status: resolved
  - key: d1-removes-vs-fenced-order
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: d1-way-forward-missing-amends-plan
    rounds: [1]
    status: resolved
  - key: d9-opaque-identity-bare-locations
    rounds: [1]
    status: resolved
  - key: stencil-decisions-no-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: d5-justification-and-duplicate-risk
    rounds: [1]
    status: resolved
  - key: amends-plan-failed-second-call-erases-first
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: amends-plan-commit-error-after-landed-commit
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-not-found-answer-falls-through
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-plan-handle-targets-never-match
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-wire-guess-misses-method-glyphs
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d7-whole-second-write-time
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: d4-shared-function-overstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orch-stencil-criterion-describes-no-change
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 2 raised 13 findings against the round-1 rewrite.
Twelve round-1 keys were not raised again and read resolved: the D2 escape bound, D3 re-sign arrow, D5 justification, D8 target carrier and offer timing, D9 trimpath paths, opaque identity and overlap attribution, D1 headroom and way-forward wording, the D7/D8 reopen bound, and the stencil acceptance criterion.
Five round-1 keys recur: the land-together key (F1, still BLOCKING, now through the never-tracked amendment record), D7 round-start comparison (F2), D7 answer-key clauses (F5), D9 per-card commit premise (F8), and the D1 fenced-order rule (F11).
New keys cover D7 failure handling (F3, F4, F10), D9 trace matching (F6, F7), D5's wire guess (F9), D4's rationale (F12) and the orch-stencil criterion (F13).
Fixes that change a decision, record location or contract are `design-wrong`; local rewordings whose text the finding fixes are `mechanical`.
