---
round: 1
ledger:
  - key: restep-and-verify-retry-warn-every-poll
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: dirty-pair-skip-info-repeats
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings on the merge-in.
F1: the parent's binary re-step check and Go verify-retry read in `battenshed` Warn on every poll, bypassing the task's `seamFailed`/`seamRecovered` episode guard.
F2: the dirty-pair skip repeats its Info line while nothing has changed.
Both fixes are local and spelled out in the finding text.
