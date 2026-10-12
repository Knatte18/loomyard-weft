---
round: 2
ledger:
  - key: d6-watchdog-false-hang-no-way-forward
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-busy-threshold-single-core-ceiling
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-enclosing-variable-trusted-unchecked
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-integration-criterion-satisfied-by-outer-watchdog
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d7-inherited-slot-exemption-disables-watchdog
    rounds: [1]
    status: resolved
  - key: d5-exit-code-128-rule-unbounded
    rounds: [1]
    status: resolved
  - key: d3-map-value-replaced-keeps-unknown-keys
    rounds: [1]
    status: resolved
  - key: d9-scan-broader-than-rule
    rounds: [1]
    status: resolved
  - key: d7-bench-fuzz-exemption-unbounded
    rounds: [1]
    status: resolved
  - key: d7-rationale-board-deliberation
    rounds: [1]
    status: resolved
  - key: missing-blank-lines-before-headings
    rounds: [1]
    status: resolved
---

Round 1's nine findings: seven were not re-raised in round 2 and are marked resolved, as is the inherited-slot exemption, whose redesign (a dedicated variable) produced round 2's F2.
Two persist: D6's way forward for a watchdog false hang, now widened to the loom and landing verify gates (BLOCKING), and D7's one-core threshold, now framed as the integration criterion sitting on the boundary.
Round 2 adds two new keys: the unchecked enclosing-gate-test variable and an integration criterion the outer watchdog can satisfy.
