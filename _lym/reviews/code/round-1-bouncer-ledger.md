---
round: 1
ledger:
  - key: heartbeat-logged-runs-unprimed-after-failed-start-read
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: template-equal-adopt-pin-dropped-and-reseeded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: adopt-file-header-names-old-test-suite
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: deps-log-doc-names-another-symbol
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, all LOW or NIT.
F1 (heartbeat priming after a failed start read) and F2 (a template-equal `adopt` pin dropped and reseeded) are `design` at LOW.
F3 (adopt.go header) and F4 (`Deps.Log` doc) are `consistency` NITs.
No finding is gating at MEDIUM or worse, so the loop converged in round 1.
