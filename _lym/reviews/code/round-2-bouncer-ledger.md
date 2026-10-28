---
round: 2
ledger:
  - key: waiterless-run-counted-live
    rounds: [1]
    status: resolved
  - key: carryover-selection-untested
    rounds: [1]
    status: resolved
  - key: path-prompt-read-as-slash-command
    rounds: [1]
    status: resolved
  - key: first-instruction-dropped-when-prompt-undelivered
    rounds: [1]
    status: resolved
  - key: carryover-failure-paths-untested
    rounds: [1]
    status: resolved
  - key: waitergone-skip-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: battenshed-stale-live-run-comments
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five `design` findings; round 2's review confirms each addressed (waiterless runs no longer counted live by the loomcli callers, carry-over selection and failure paths pinned, path-opening prompts kept, undelivered first instruction kept).
Round 2 raised two non-gating findings.
F1 is the missing test of the `WaiterGone` skip in `liveRunStrands` and `agentActivityFor`.
F2 is stale "live run" wording in battenshed doc comments.
