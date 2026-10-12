---
round: 2
ledger:
  - key: import-allowlists-not-edited
    rounds: [1]
    status: resolved
  - key: orch-watcher-late-start-interrupt
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: prime-network-gate-slug-empty
    rounds: [1]
    status: resolved
  - key: shedverbs-doc-bare-pause-stale
    rounds: [1]
    status: resolved
  - key: pause-status-omits-network-ops
    rounds: [1]
    status: resolved
  - key: arbiter-clause-false-bound
    rounds: [1]
    status: resolved
  - key: card12-impact-undercount
    rounds: [1]
    status: resolved
  - key: orch-start-under-entry-interrupted
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: sandbox-s11-collision
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: batten-gate-slug-call-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised seven findings; round 2 does not raise six of them again, and the review's rejected list covers the import allowlists, so those are resolved.
F1 of round 2 (BLOCKING, design) is the same area as the round-1 orch watcher key, now as a live-watcher case under grace 0, so that key is reopened at BLOCKING.
F2 (`orch-start-under-entry-interrupted`), F3 (`sandbox-s11-collision`) and F4 (`batten-gate-slug-call-untested`) are new in round 2.
No finding was relabelled.
