---
round: 1
ledger:
  - key: upsert-kind-flip-note-to-task-keeps-revised
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: formatage-boundaries-and-clamp-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two LOW findings.
F1 maps to `upsert-kind-flip-note-to-task-keeps-revised`: `stampDates` exempts every note-to-task kind change from revision, not only `promote`.
F2 maps to `formatage-boundaries-and-clamp-untested`: `FormatAge` unit boundaries and its clamp have no unit test.
The previous ledger was `(none)`, so nothing carries forward.
