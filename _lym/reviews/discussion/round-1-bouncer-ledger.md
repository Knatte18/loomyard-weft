---
round: 1
ledger:
  - key: round-checks-inside-strict-parse
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: origin-parent-exemption-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fixerless-round-indistinguishable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: carries-bypasses-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parent-answered-decision-reescalates
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fix-check-cap-behaviour-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: post-fix-target-id-not-recorded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stalemate-vs-softcap-autoaccept
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stalemate-arbiter-eligibility
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fix-regression-untouched-file-refused
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-completeness-vs-open-risk
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: amendment-rounds-full-ambiguity
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: decision-exclusion-assumes-routing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-acceptance-omits-repeats
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 14 findings over the decision record, none carried from an earlier round.
Keys map to F1 through F14 in review order.
The two BLOCKING findings break D4's own guarantees: the cap-passed review the strict parse rejects, and the unbounded origin parent exemption.
The MEDIUM findings are unstated behaviours at the seams between D1, D2, D3, D5 and D6 and the existing Bouncer, arbiter and fix paths.
