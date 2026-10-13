---
round: 2
ledger:
  - key: d11-audit-table-incomplete
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: recovery-branches-unreached-by-tests
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

`d11-audit-table-incomplete` recurs and has escalated to BLOCKING.
Round 1's fix added rows for unnamed loops but kept the method of reading lines lexically inside loop bodies.
Round 2 found per-iteration Warn lines in callees and callbacks (reed layout and name-repair paths, loomcli's loop wake hook, landingshed's parent-lock wait).
The fix adds cards and verdict rows, so it needs a fresh review.

`recovery-branches-unreached-by-tests` recurs at MEDIUM.
Round 1's fix added Recover branches to card 10 whose pending-send and startup-dismiss paths have no reaching test.
The remedy extends test rows already named in the card, so it is mechanical.
