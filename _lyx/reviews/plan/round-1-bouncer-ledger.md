---
round: 1
ledger:
  - key: card2-drops-per-row-guardrail-comments
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: testchecktexts-steer-linebreak-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: invalid-config-way-forward-misses-local-layer
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card5-uses-unexported-checkload
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, none gating.
F1 (card 2 guardrail comments) and F2 (`TestCheckTexts` steer set) are MEDIUM in non-gating classes.
F3 (way forward cannot clear a local-layer key) is a LOW design finding, and F4 (card 5 `Uses` entry) is a NIT.
Each fix is a local text correction the finding already fixes.
