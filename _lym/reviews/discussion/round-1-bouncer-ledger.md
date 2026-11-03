---
round: 1
ledger:
  - key: d1-add-inputs-refuse-live-pair
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-land-way-forward-does-not-recover
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-push-departs-push-both-sides
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: d1-reconcile-change-underspecified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-permanent-transient-split-by-exclusion
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-cancel-at-land-misreads-childreachedparent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-cancel-lifts-owner-pause
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-lease-tip-disagrees-with-d4-push
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-sweep-has-no-origin-tip-archive
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risks-defer-research
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-bound-misses-surviving-local-weft
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d4-diverged-tip-not-fetched
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-rule-reads-only-remote-tip
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d7-bound-silent-on-longer-hold
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d7-criterion-pins-momentary-state
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-adoptlocal-rule-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists to carry forward.
All 16 findings of the round-1 review are open.
Three are BLOCKING: the adopt-only `Add` inputs (F1), the Land way forward (F2), and D4's departure from `PATTERN-push-both-sides` (F3).
Six more are MEDIUM and gating (F4 to F9), and the Open risks finding is MEDIUM too (F10).
Findings F12, F15 and F16 are local corrections the review text already fixes, so they are marked mechanical.
The others change a decision, a bound or a rule and need a fresh review.
