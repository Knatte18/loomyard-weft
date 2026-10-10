---
round: 1
ledger:
  - key: git-config-flag-row-misses-flag-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: pathless-violation-classes-hand-copy
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: redundant-glob-operand-readonly-lyx-segment
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised three findings.
F1 is a design gap: the `git config --get --file f k` test row is rejected by the argument-count bound, so it does not pin the flag check.
F2 is a hand-copied class list in `recordbatch.go` with only one exercised class.
F3 is a redundant `word.glob ||` operand in `readOnlyLyxSegment`.
The remaining plan items (cards 1 to 4 and 6 to 8, docs, budget) showed no defect.
