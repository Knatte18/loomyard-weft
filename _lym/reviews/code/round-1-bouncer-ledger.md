---
round: 1
ledger:
  - key: accept-name-match-admits-non-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: accept-verb-orchestration-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: accepted-output-dropped-before-move
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: accept-concurrent-accepting-dir
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: accept-stale-run-status
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-capture-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opened six keys, none carried from an earlier round.
F1 maps to `accept-name-match-admits-non-test` and F2 to `accept-verb-orchestration-untested`; both are gating.
F3 to F6 map in order to `accepted-output-dropped-before-move`, `accept-concurrent-accepting-dir`, `accept-stale-run-status` and `stale-capture-wording`.
F4 adds a lock to the accept path, which is new design text, so it is marked `design-wrong`; the others are local corrections the review text already fixes.
