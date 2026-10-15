---
round: 1
ledger:
  - key: keepviewer-persistence-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: `Display.KeepViewer` persists as `keepViewer,omitempty`, and `types_test.go` pins neither the omission when false nor the round trip.
The fix extends an existing test and settles the finding's text locally.
