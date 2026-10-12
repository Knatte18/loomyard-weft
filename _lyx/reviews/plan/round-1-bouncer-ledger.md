---
round: 1
ledger:
  - key: import-allowlists-not-edited
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: orch-watcher-late-start-interrupt
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: prime-network-gate-slug-empty
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: shedverbs-doc-bare-pause-stale
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: pause-status-omits-network-ops
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: arbiter-clause-false-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card12-impact-undercount
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised seven findings (F1 to F7), mapped in order to the keys above.
Gating: F1 (BLOCKING, design) and F2 (MEDIUM, design).
No earlier ledger existed, and no finding was relabelled.
