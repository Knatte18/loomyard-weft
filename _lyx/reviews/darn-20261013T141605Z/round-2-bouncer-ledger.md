---
round: 2
ledger:
  - key: pack-in-separate-file-not-last-section
    rounds: [1]
    status: resolved
  - key: summary-describes-abandoned-pack-file-design
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: promptpath-computation-moved-needlessly
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised the design finding that the pack lived in a separate file with a pointer line; the round 1 fix rendered it inline as the prompt's last section, and round 2 did not re-raise it.
Round 2 raised two non-gating leftovers of the abandoned pack-file design.
The change description still describes that design, and `promptPath` in `BeginBatch` was moved above code that no longer needs it.
Both fixes are local and fully specified by the finding text.
