---
round: 1
focus:
  - "Fidelity: check that each of D1–D7 and every Constraint in the decision record is carried by some card, and that cards 7 and 8 introduce no scope the record does not license."
  - "Attack surface: cards 5, 10 and 11 add a wait, a window-opening op and a `--window` flag; check whether each `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Attack surface: card 3 and card 6 change when a driver messages its parent; check whether the Intent states what a suppressed notice can hide and what bounds it."
  - "Granularity: cards 9 and 10 (reed) and cards 1 and 2 (orch) may be one unit split in two; check whether each card is an independently reviewable unit."
  - "Test economy: for each card planning a new test, name the existing test that already covers the behavior, especially for cards 4, 5, 7 and 8."
  - "Verify: check that every package a card targets has its tests run by `## verify:`, including any `-tags integration` or `-tags tmux` tests, and that the section follows the template order."
  - "ImpactSummary: on Edit and Delete cards, check that each one states a real blast-radius conclusion rather than restating Intent."
---

Focus set from the plan overview and the section list of the decision record.
