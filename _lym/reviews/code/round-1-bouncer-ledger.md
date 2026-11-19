---
round: 1
ledger:
  - key: unmarked-exit-cause-wallclock-boot-time
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reconciliation-mark-hides-unmarked-exit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: end-without-envelope-exit-code-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lock-loser-mark-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: doc-pid-file-shape-omits-new-field
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: doc-early-end-sources-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none carried over, as this is the first round.
F1 (the wall-clock boot-time comparison) and F2 (a reconciliation mark counted as exit evidence) gate at MEDIUM design.
F1's fix swaps the pid record's start time for a boot id and changes an interface, so it needs a fresh review.
F2's fix is local to `loopExitEvidence` and its tests.
F3 and F4 are test gaps, and F5 and F6 are doc corrections; F5's wording depends on how F1 lands.
