---
round: 1
ledger:
  - key: fix-table-missing-segment-color
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: deleted-tests-listed-as-bare-file-path
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card6-misses-test-comments-naming-retired-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 is the first round; there was no prior ledger.
F1 maps to `fix-table-missing-segment-color`: card 2's `fixTable` omits the segment color, a `PATTERN-spawn-color` breach.
F2 maps to `deleted-tests-listed-as-bare-file-path`: cards 4 and 10 list bare test-file paths where Delete glyphs belong.
F3 maps to `card6-misses-test-comments-naming-retired-rows`: test comments naming retired rows sit outside every card.
