---
round: 2
ledger:
  - key: stale-whole-file-read-bound-missing
    rounds: [1]
    status: resolved
  - key: open-risks-measure-bullet-belongs-to-board-entry
    rounds: [1]
    status: resolved
  - key: auto-mode-bullet-points-at-support-log
    rounds: [1]
    status: resolved
  - key: located-line-numbers-escape-stale-read-guard
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: open-risks-bullet-omits-card-files
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1's three findings are resolved: round 2 does not raise them, and its review states the stale-read bound on Decision 2's batch-wide reads is settled.
Round 2 raised two new findings.
F1 (design, LOW): line numbers from locating an unresolved glyph are taken before the first edit, and Decision 5's guard covers only the pack's listed lines.
F2 (consistency, NIT): Open risks bullet 1 omits the card files that Decision 1 reads in the same phase.
