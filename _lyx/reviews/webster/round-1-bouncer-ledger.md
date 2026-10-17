---
round: 1
ledger:
  - key: reseed-after-publish-strands-own-pr
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: loomcli-doc-omits-run-start-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-row-pins-counted-note-edit-moves-snapshot
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, none carried from earlier rounds.
F1 (`reseed-after-publish-strands-own-pr`) is the gating one: the way forward in `wireLanding` omits a consequence, and its fix is a string clause plus an assertion, so it is mechanical.
F2 and F3 are non-gating and mechanical.
