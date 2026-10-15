---
round: 3
ledger:
  - key: repeated-conflict-halt-no-way-forward
    rounds: [1]
    status: resolved
  - key: stale-conflict-record-routes-other-stuck
    rounds: [1]
    status: resolved
  - key: rerouted-round-inherits-hard-cap
    rounds: [1]
    status: resolved
  - key: repeat-check-comparison-unspecified
    rounds: [1]
    status: resolved
  - key: d1-rebegun-batch-wording
    rounds: [1]
    status: resolved
  - key: reroute-cap-halt-wrong-way-forward
    rounds: [2]
    status: resolved
  - key: last-routed-written-before-reroute-taken
    rounds: [2]
    status: resolved
  - key: d2-record-existence-overclaim
    rounds: [2]
    status: resolved
  - key: d3-bullet-list-swallows-decision
    rounds: [2]
    status: resolved
  - key: d2-removal-before-run-lock-and-failure-unstated
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised nine findings; round 3's focus verdicts show the reroute-cap way forward, the last-routed timing, the record-lifecycle claim and the D3 list structure are sound, so all nine are resolved.
Round 3 raised one new finding, F1: D2 places the Run-entry record removal before the run lock and states no behavior for a failed removal.
The finding text fixes the wording change, so the fix is mechanical.
