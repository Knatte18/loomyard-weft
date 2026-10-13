---
round: 1
ledger:
  - key: step-after-goto-row-misses-conjunct
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: tackler-test-dead-spawn-column
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: peers-row-unreachable-warn-fallback
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all with fixes the review text already fixes.
F1 (card 2): the "moved by a goto and then stepped" row is refused by the producer-is-target conjunct, so no row isolates the every-entry-is-a-goto conjunct.
F2 (card 2): the tackler test gains a `spawn` column no row uses.
F3 (card 4): a `logger.Warn` fallback in `hubPeersDeps` has no reaching input or test.
