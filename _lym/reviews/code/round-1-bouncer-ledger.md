---
round: 1
ledger:
  - key: refscanner-gate-blanking-bounds-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: accept-audit-way-forward-refusal-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-segment-ends-at-redirect-ampersand
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cardid-finding-text-and-summary-drift
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: batch-fabric-evidence-nil-guard-untested
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised five low-weight findings (F1 to F5 in order above): two test-coverage gaps on the scanner and the way-forward refusals, one redirect-ampersand false positive, one message and summary drift, and one untested nil guard.
None is gating, so the loop converges.
