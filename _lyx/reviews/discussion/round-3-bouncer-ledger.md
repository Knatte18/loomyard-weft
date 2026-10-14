---
round: 3
ledger:
  - key: partial-apply-unrecoverable
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: raw-io-row-does-not-cover-rename
    rounds: [2]
    status: resolved
  - key: journal-omits-move-destinations
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: journal-write-failure-no-outcome
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: raw-io-row-omits-journal
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eight findings and round 2 resolved seven of them.
The surviving thread was D2's failure path: round 2 found the mid-apply reversal's halt unrecoverable, its source and order unstated, and the raw-I/O row silent on rename.
Round 3 finds those closed: the renumber journal supplies the pre-call bytes, the apply order is sound, and the row names rename and the plan files.
Three gaps remain, all in the new journal text: the created-name list omits move destinations (F1), the journal's own write and an undecodable journal have no outcome (F2), and the raw-I/O row omits the journal (F3, LOW).
F1 and F2 are first raised this round, so Go carries them into `## Open risks`.
