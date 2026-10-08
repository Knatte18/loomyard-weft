---
round: 2
ledger:
  - key: plan-gate-only-bound-not-implementable-on-shared-seams
    rounds: [1]
    status: resolved
  - key: d1-first-card-depends-on-later-checks
    rounds: [1]
    status: resolved
  - key: no-mechanism-turns-exposed-defect-into-card
    rounds: [1]
    status: resolved
  - key: d5-reference-scan-limits-unstated
    rounds: [1]
    status: resolved
  - key: resign-arrow-head-unbounded
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: no-double-report-rule-delete-only
    rounds: [1]
    status: resolved
  - key: goal-overclaims-verdict-identity
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d2-bound-names-wrong-sequencing-unit
    rounds: [1]
    status: resolved
  - key: restatements-outside-support-log
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: open-risks-promises-unbuilt-rubric-backstop
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-stability-names-wrong-transition
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-d6-both-claim-stencil-edit
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's gating core (plan-gate-only bound, D1 card ordering, defect-to-card path, reference-scan limits) did not reappear in round 2 and is marked resolved.
Three round-1 themes persist at lower severity: the re-sign arrow's check placement and handle target (F3), the Goal's over-claimed verdict identity (F4), and restatements outside the support log (F6).
Round 2 adds two MEDIUM design findings: Open risks cites a Plan-Review rubric backstop that D6 never builds (F1), and D3's stability claim and criteria name `CanonicalizeHandles` where `BindHandles` at record-batch is the real transition (F2).
F5 is a new LOW consistency conflict over which card removes the stencil sentence.
