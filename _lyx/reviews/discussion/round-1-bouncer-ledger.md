---
round: 1
ledger:
  - key: partial-apply-unrecoverable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: slug-bound-cites-missing-rule
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: begun-card-bound-without-lease
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: intent-flag-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: rebaseline-open-risk-answerable
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: wrong-run-record-reading-named
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: reachability-rule-overreaches-io-errors
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: residual-risk-bullet-belongs-in-support-log
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eight findings, all open.
The gating ones concern D2's new rename and delete edges: a mid-apply failure has no way forward (F1), the slug bound rests on a rule the plan format lacks (F2), and the begun-card check is not held under webster's state-mutation lease (F3).
The LOW findings are an unbounded `--intent` value, a rebaseline question the code already answers, a mislabeled run-record reading, an over-wide reachability rule, and a residual-risk bullet that belongs in the support log.
