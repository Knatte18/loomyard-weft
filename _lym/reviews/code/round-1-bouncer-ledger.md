---
round: 1
ledger:
  - key: land-refusal-ends-episode
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: beam-land-gate-seams-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: markready-commit-failure-uncommitted-flag
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ready-refusal-order-and-errors-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: pawlrecheck-ready-line-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: tick-comment-stale-notice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loomcli-doc-omits-ready-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, all open.
The gating pair is `land-refusal-ends-episode` (F1) and `beam-land-gate-seams-untested` (F2).
The rest are LOW: a commit-failure gap in `MarkReady` (F3), two test-pinning gaps (F4, F5), and two stale docs (F6, F7).
