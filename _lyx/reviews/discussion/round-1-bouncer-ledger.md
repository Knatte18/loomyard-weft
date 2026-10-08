---
round: 1
ledger:
  - key: plan-gate-only-bound-not-implementable-on-shared-seams
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d1-first-card-depends-on-later-checks
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: no-mechanism-turns-exposed-defect-into-card
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d5-reference-scan-limits-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: resign-arrow-head-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: no-double-report-rule-delete-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: goal-overclaims-verdict-identity
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-bound-names-wrong-sequencing-unit
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: restatements-outside-support-log
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 raised nine findings against the decision record, none recurring because this is the first round.
The gating core is the plan-gate-only bound: the shared `planparser` and `resolvePass` seams run at dispatch too, so D3 and D5 cannot keep their new checks out of `ValidateDispatch` as written.
D5 also lacks bounds for its tree-wide reference scan and for the re-sign arrow's head, and D1's first card depends on checks that later cards build.
The remaining findings are wording and relocation cleanups.
