---
round: 1
ledger:
  - key: plain-promote-labelled-note-missing-record-dead-end
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1.
A plain promote of a labelled discussed note whose record file is missing gets no recipe raddle, so the label rule refuses it with a way forward naming the same plain promote.
The finding's fix text names the exact condition change, the test row and the comments to update, so a fixer can apply it without a design decision.
