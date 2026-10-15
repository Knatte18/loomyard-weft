---
round: 1
ledger:
  - key: stale-retired-config-key-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-overview-trims-outside-cards
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings and no gating one.
F1 (`stale-retired-config-key-comments`): three comments still name retired config keys and fields; a comment-only fix.
F2 (`pattern-overview-trims-outside-cards`): card 2 trimmed clauses from five PATTERN entries to make room for a new one; the background files still carry the rules, and the default is to keep the trims and record the choice.
