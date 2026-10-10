---
round: 1
ledger:
  - key: log-level-env-reset-by-root-hook
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: boot-refusals-unreached-at-planned-site
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
---

Round 1 raised two findings.
F1 maps to `log-level-env-reset-by-root-hook`: the root hook's `SetVerbosity` with the `-v` count resets the level `LYX_LOG_LEVEL` set, so the documented Debug route for the detached daemon does not work.
F2 maps to `boot-refusals-unreached-at-planned-site`: the socket-path and shell refusals in `ensureServerAndSessionLocked` are shadowed by a duplicate call in `zoom.go`, and no test reaches the planned site.
