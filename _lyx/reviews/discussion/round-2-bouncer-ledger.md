---
round: 2
ledger:
  - key: partial-apply-unrecoverable
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: slug-bound-cites-missing-rule
    rounds: [1]
    status: resolved
  - key: begun-card-bound-without-lease
    rounds: [1]
    status: resolved
  - key: intent-flag-unbounded
    rounds: [1]
    status: resolved
  - key: rebaseline-open-risk-answerable
    rounds: [1]
    status: resolved
  - key: wrong-run-record-reading-named
    rounds: [1]
    status: resolved
  - key: reachability-rule-overreaches-io-errors
    rounds: [1]
    status: resolved
  - key: residual-risk-bullet-belongs-in-support-log
    rounds: [1]
    status: resolved
  - key: reversal-source-and-apply-order-unstated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: raw-io-row-does-not-cover-rename
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
---

Round 1 raised eight findings; round 2 resolved seven of them, with the slug, intent and begun-card bounds confirmed as not defects.
The surviving thread is D2's failure path.
Round 1 found a mid-apply failure unrecoverable.
D2 now reverses it, but round 2 finds the reversal-failure halt names a hand reversal that is no admitted way forward (F1, BLOCKING).
Two further gaps sit beside it: the reversal needs pre-call bytes and a stated create and delete order (F2), and the spec's raw I/O row does not cover a rename or plan files (F3).
