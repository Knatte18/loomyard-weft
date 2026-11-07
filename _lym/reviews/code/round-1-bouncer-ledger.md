---
round: 1
ledger:
  - key: constructors-transcript-wait-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: state-read-order-and-sidechain-first-tail-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: polling-past-first-reread-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: first-tail-check-silences-threshold-crossing-call
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: derived-two-second-tally-in-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all with fixes the review text fully specifies.
F1 (the constructors' `transcriptWait` unpinned) is the only gating one.
F2 and F3 are missing test rows, F4 is a doc statement of the first-tail reading's consequence, and F5 is a derived tally in a comment.
