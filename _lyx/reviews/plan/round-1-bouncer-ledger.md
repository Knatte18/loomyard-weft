---
round: 1
ledger:
  - key: chair-settle-wait-unbounded-for-written-off-advisor
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised one finding (F1): the chair-batches card's rewritten settle bullet waits on every advisor's answer but the Intent names no bound for an advisor that was written off, failed or never started.
The reviewer proposes extending the Intent so such an advisor counts as giving no answer, bounded by the write-off bullet and the seat directive.
