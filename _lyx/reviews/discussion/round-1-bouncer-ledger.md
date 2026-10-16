---
round: 1
ledger:
  - key: pattern-declaration-unbounded-guard-downgrade
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: verify-fix-departure-bypasses-go-route
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: uncaused-failure-bound-false-verify-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: spawn-refusal-names-diagnostic-not-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: no-rewrite-tool-exception-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-out-carries-requirement-and-narrative
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger exists.
Two BLOCKING design findings stand: a declared PATTERN change disarms the PATTERN guard for its entry with no stated bound or landing check (F1), and D2.8 routes departures through the fork's raw reply against the operator's Go-route ruling (F2).
F3 is a design MEDIUM: D2.6's bound assumes the plan-level verify gate runs the whole suite.
F4 (refusal way forward and burler's missing spec section), F5 (D1.4 wording) and F6 (Scope Out narrative and a missing Decision sentence) are consistency findings.
