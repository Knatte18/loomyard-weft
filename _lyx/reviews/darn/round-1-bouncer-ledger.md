---
round: 1
ledger:
  - key: claudeengine-doc-open-bypass-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: list-models-registry-load-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
F1 is a stale sentence in the `claudeengine` package doc about when the open launch carries the bypass flag.
F2 is a missing test for the `--list-models` registry-load failure.
Neither is design class, so the loop converges.
