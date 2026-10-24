---
round: 1
ledger:
  - key: unarmed-stall-guard-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: local-no-trace-file-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: trace-cleanup-retry-warn-override-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW test-coverage gaps in `TestRun_RemoteBounds`: the unarmed stall guard (F1), the absence and presence of `GIT_TRACE_PACKET` for local and remote commands (F2), and trace cleanup, the retry Warn and the operator override (F3).
No design finding was raised.
