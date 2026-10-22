---
round: 3
ledger:
  - key: amends-plan-commit-failure-land-together
    rounds: [1, 2]
    status: resolved
  - key: d9-trimpath-panic-paths-dropped
    rounds: [1]
    status: resolved
  - key: d9-no-per-card-commit
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d8-amendment-target-carrier
    rounds: [1]
    status: resolved
  - key: d7-d8-reopen-and-widening-unbounded
    rounds: [1]
    status: resolved
  - key: d7-removal-time-comparison
    rounds: [1, 2]
    status: resolved
  - key: d7-answer-key-clauses-disagree
    rounds: [1, 2]
    status: resolved
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
    status: resolved
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
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: amends-plan-commit-error-after-landed-commit
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-not-found-answer-falls-through
    rounds: [2]
    status: resolved
  - key: d9-plan-handle-targets-never-match
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d5-wire-guess-misses-method-glyphs
    rounds: [2]
    status: resolved
  - key: d7-whole-second-write-time
    rounds: [2]
    status: resolved
  - key: d4-shared-function-overstated
    rounds: [2]
    status: resolved
  - key: orch-stencil-criterion-describes-no-change
    rounds: [2]
    status: resolved
  - key: d7-answer-key-carry-test-done-and-renumbered-cards
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
---

Round 3 raised five findings, none BLOCKING and none design-class.
Every round-2 design-class key, including the BLOCKING land-together key, was not raised again and reads resolved, as do the round-2 consistency and NIT keys not raised.

Mapping of the five findings:

- F1, the answer-key carry test for done and renumbered cards, is a new key.
- F2, `not_found` matching for member-glyph targets, sits on `d9-plan-handle-targets-never-match`, MEDIUM decision.
- F3, the landed-commit test reading the amendment record alone, sits on `amends-plan-commit-error-after-landed-commit`, LOW consistency.
- F4, the findings refusal leaving the amendment record unrestored, sits on `amends-plan-failed-second-call-erases-first`, LOW consistency.
- F5, the range rule over unreachable batch states, sits on `d9-no-per-card-commit`, LOW decision.

Open keys are a convergence-carried residue for a later check; none is gating.
