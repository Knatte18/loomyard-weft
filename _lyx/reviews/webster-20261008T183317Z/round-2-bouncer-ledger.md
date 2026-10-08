---
round: 2
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
    status: open
    class: design
    severity: MEDIUM
  - key: send-never-appeared-attempt-count
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: send-not-landed-comments-narrow
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: send-window-key-comments-engine-binding
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1's five keys were fixed and none regressed, per round 2's review.
Round 2 raised four new findings, F1 to F4, mapped in order to the last four keys.
F1 is the one gating finding: the fix for the dropped reclaim Warn made `stopLeftoverStrand` warn and stop at sites that are not a respawn.
F2 to F4 are comment and message accuracy items that do not gate.
