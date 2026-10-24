---
round: 1
ledger:
  - key: runcorewithin-doc-omits-optional-locks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: remote-optional-locks-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW non-gating findings.
F1: the `runCoreWithin` doc comment omits the `GIT_OPTIONAL_LOCKS` handling.
F2: no test pins the remote spawn path's `GIT_OPTIONAL_LOCKS=0`.
Both fixes are local and spelled out in the finding text.
