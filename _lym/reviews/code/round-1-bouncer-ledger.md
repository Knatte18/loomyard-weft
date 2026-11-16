---
round: 1
ledger:
  - key: runproc-hubowner-subtest-name-stale-testmap
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: the `HubOwner` subtest name in `internal/runproc/end_test.go` still names TestMap after the merge removed the role.
It is a local rename whose result the finding text fixes.
