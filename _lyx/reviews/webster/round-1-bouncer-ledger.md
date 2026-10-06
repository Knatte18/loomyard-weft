---
round: 1
ledger:
  - key: comment-line-breaks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: markdown-line-breaks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two LOW consistency findings, both about semantic line breaks.
`comment-line-breaks` is F1, new Go comments that keep several clauses or sentences on one line.
`markdown-line-breaks` is F2, new markdown that joins independent clauses on one line.
Neither is gating, and no fresh reviewer has seen the fixes yet, so both stay open.
