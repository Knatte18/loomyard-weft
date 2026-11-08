---
round: 1
ledger:
  - key: forward-empty-body-posts
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1, keyed `forward-empty-body-posts`.
A forwarded message with empty text and no files posts an empty body, which creel refuses and the read retries.
The fix is local: skip such a message in `forward` and add one table row to `TestInboundRead`.
It is LOW, so it does not block convergence.
