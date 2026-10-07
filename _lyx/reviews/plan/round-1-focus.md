---
round: 1
focus:
  - "Fidelity: walk D1 through D14 and the decision record's Constraints and settle which card carries each; in particular D7 (note evidence and history truncation), D9's `doc.go` update, D13's doc, stencil and help-text updates, and D12's `ParseReview` call sharing between gate and verb."
  - "Attack-surface: for each card adding an Enter retry (3), a strand removal (5, 6, 7), a halt skip (4), a wait (16), or a gate re-prompt (17, 18), settle whether its `**Intent:**` states what it can skip or remove and what bounds it, as the matching decision's Bound clause does."
  - "Verify coverage: settle whether every package a card targets has its tests run by `## verify:`, including packages named by cards 12, 13, 15, 16, 17 and 18."
  - "Test economy: for cards 3, 6, 7, 14, 15, 16 and 17, settle whether each plans a new top-level test where an existing or prune-folded test, scenario or table row already covers the path, and name that test."
  - "Granularity: settle whether the split of cards 10/11 and 12/13, and of 17/18, follows independently reviewable units rather than single symbols."
  - "Scope: settle whether any card adds a verb, flag or config key beyond D12's self-check verb, or touches the out-of-scope orch resume verb."
---

Seed focus only; the rubric's Do-not-flag list was checked against each entry.
