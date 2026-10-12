---
round: 1
ledger:
  - key: overlay-canonicalization-writes-and-reloads
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: plan-gate-double-delete-before-reference
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: recover-batch-refusal-undecided
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: candidate-walk-skipped-directories
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: overlay-accepts-non-plan-md
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: missing-acceptance-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: bisection-procedure-unspecified
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: no-enclosing-span-disposition
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised eight findings, none carried from earlier rounds.
The gating pair is the overlay path that writes plan files through handle canonicalization and then reloads from disk, and the two delete-before-reference forms that both run at the plan gate once Webster has done cards.
The rest are bounded gaps: recover-batch's refusal, skipped directories in the candidate walk, non-plan `.md` overlay names, two missing acceptance rows, the bisection procedure, and the no-enclosing-span case.
