---
round: 1
ledger:
  - key: d7-failed-remote-update-and-diverged-state
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-lease-vs-ancestry-check-tip
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d6-stuck-terminal-status-omitted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-relaxation-let-through-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: dispatch-check-card-scope-inconsistent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d6-recover-batch-refuses-uncheckable-failed-batch
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d9-q11-dependence-no-fallback
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

First round, no previous ledger.
Seven findings are open, all against `decision-record.md`.
The BLOCKING one is D7's failure semantics: a failed remote update after the local reset, and the already-diverged state the bug reported.
The other gating ones are D6's missing `stuck` status and D3's unstated let-through.
