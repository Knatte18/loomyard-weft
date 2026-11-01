---
round: 1
ledger:
  - key: gated-first-row-loses-current-producer
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rowat-empty-producers-untested
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised two findings, both below the gating threshold.
F1 maps to `gated-first-row-loses-current-producer`: the post-gate status re-read in `stepLocked` drops the resolved first row, a latent case since no current recipe gates its first row.
F2 maps to `rowat-empty-producers-untested`: a missing test row for `RowAt`'s empty-producers branch.
