---
round: 1
ledger:
  - key: d7-inherited-slot-exemption-disables-watchdog
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-busy-threshold-single-core-ceiling
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-watchdog-false-hang-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-exit-code-128-rule-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-map-value-replaced-keeps-unknown-keys
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d9-scan-broader-than-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d7-bench-fuzz-exemption-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d7-rationale-board-deliberation
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: missing-blank-lines-before-headings
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised nine findings, all open.
The gating ones concern D7's watchdog (inherited-slot exemption, one-core threshold), D6's refusal wording for a watchdog false hang, and D5's exit-code rule.
