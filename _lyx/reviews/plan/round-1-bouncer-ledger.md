---
round: 1
ledger:
  - key: begin-batch-doc-comment-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1, mapped to `begin-batch-doc-comment-stale`.
Card 3 edits `BeginBatch` but leaves its doc comment claiming a pack always means a pack file.
The fix is a sentence in card 3's Intent, so it is mechanical.
