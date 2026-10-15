---
round: 1
ledger:
  - key: protocol-read-all-assertion-vacuous
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: read-all-from-combination-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: store-read-doc-comment-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all against card 17.
F1 (protocol-read-all-assertion-vacuous) is the gating one.
F2 and F3 are local wording corrections to the card's Intent.
