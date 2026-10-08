---
round: 1
ledger:
  - key: per-check-process-costing-worktree-resolution
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: down-reads-reed-state-for-reap-roots
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: running-arm-wait-reason-says-sleeping-before-bounce
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: watched-stencil-names-who-announces-stops
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: changed-comments-column-wrapped-and-pattern-text-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 opened five findings and resolved none.
The gating pair is the per-check `taskWorktreeLocation` cost behind `ReadDecision`, `StopReport` and `AttachDir` (BLOCKING), and `Down` loading reed state to find its reap roots (MEDIUM).
The other three are a stale wait-reason string, a stencil that describes who announces stops, and comment line breaks plus a stale pattern bullet.
