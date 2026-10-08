---
round: 4
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
    status: resolved
  - key: send-first-enter-past-window
    rounds: [3]
    status: resolved
  - key: run-test-gofmt-drift
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 and 2 keys were resolved by round 3.
Round 3's two open keys, `stop-guid-refusals-untested` and `send-first-enter-past-window`, are fixed in code and not re-raised in round 4, so both are resolved.
Round 4 raised one NIT, F1, mapped to `run-test-gofmt-drift`: two `run_test.go` table rows the branch left unaligned for `gofmt`.
It is a `consistency` finding and does not gate.
