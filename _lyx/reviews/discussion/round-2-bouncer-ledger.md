---
round: 2
ledger:
  - key: d7-followup-card-needs-overview-index
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d1-report-head-lock-bound-by-trust
    rounds: [1]
    status: resolved
  - key: d6-card-amended-mark-lost-on-failed-recovery
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: d3-archive-only-guards-unstated
    rounds: [1]
    status: resolved
  - key: d3-d4-reset-start-tree-disagree
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d10-dead-remedy-contradicts-master-ladder
    rounds: [1]
    status: resolved
  - key: d10-config-invalid-unbounded
    rounds: [1]
    status: resolved
  - key: d12-migration-both-keys-and-key-levels
    rounds: [1]
    status: resolved
  - key: scope-d8-acceptance-disagree
    rounds: [1]
    status: resolved
  - key: d13-numeric-inputs-no-usable-source
    rounds: [1]
    status: resolved
  - key: auto-mode-narrative-and-open-risk-misplaced
    rounds: [1]
    status: resolved
  - key: d7-begun-index-line-no-mechanism
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
---

Round 2 raised four findings, F1 to F4.
F1 (BLOCKING, `design`) is the round-1 D7 follow-up-card finding again: the index-only landing path is right in shape, but the second hash covers the framing instead of everything outside the Card Index.
F2 is new and sits in the same decision: the begun-card index line has no mechanism beyond the id-only card-set rule.
F3 continues the D3/D4 reset-tree mismatch, now narrowed to the missing archive-only row.
F4 continues the D6 `card_amended` lifetime, now narrowed to a repeat amendment of an already rendered card.

Eight round-1 keys produced no round-2 finding and are marked resolved.
The review's focus verdicts name D1, D12, D6's core mark lifetime and the D3/D4 guard order as not defects.
