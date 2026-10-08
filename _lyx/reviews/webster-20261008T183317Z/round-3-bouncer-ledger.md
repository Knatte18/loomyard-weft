---
round: 3
ledger:
  - key: send-retry-loops-count-uncapped
    rounds: [1]
    status: resolved
  - key: webster-reclaim-warn-dropped
    rounds: [1]
    status: resolved
  - key: send-config-comments-drift
    rounds: [1]
    status: resolved
  - key: send-never-typed-message
    rounds: [1]
    status: resolved
  - key: wait-header-stale-abandonstartup
    rounds: [1]
    status: resolved
  - key: webster-stop-leftover-overbroad
    rounds: [2]
    status: resolved
  - key: send-never-appeared-attempt-count
    rounds: [2]
    status: resolved
  - key: send-not-landed-comments-narrow
    rounds: [2]
    status: resolved
  - key: send-window-key-comments-engine-binding
    rounds: [2]
    status: resolved
  - key: stop-guid-refusals-untested
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
  - key: send-first-enter-past-window
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 2's four open keys are not re-raised in round 3 and are resolved.
Round 3 raised two new findings.
F1 maps to `stop-guid-refusals-untested`, a test-coverage gap in the guid form of the stop verb; its class is `scope`, so it does not gate.
F2 maps to `send-first-enter-past-window`, a LOW `design` finding about an Enter sent after the submit window closed.
