---
round: 1
focus:
  - "Shared Decision on gate spans: check in shuttle's wait loop and in `shedengine` that spans are measured on the run clock only there, stored as a plain stdlib struct of RFC 3339 strings, and that no two spans of one entry can overlap."
  - "Shared Decision D1: check every closed span is drawn with `statuspane.FormatSeconds` and every time that includes the open visit with `statuspane.FormatMinutes`, in the checklist, round totals and gate lines."
  - "Entry numbers: check the checklist and batten's overview stage cell both take numbers from `loomrecipe.EntryNumbers` alone, and that the rows lookup covers every recipe including batten's."
  - "Carry of gate count and spans (cards 4, 5, 6): check every gated row, and PR-Rework's Done and Stuck exits, copy both into the history entry, and that the shed history folds append spans rather than overwrite them."
  - "Role prefixes (cards 13, 14): check roles travel as role strings and only `agentname`, through reed, forms a session name, and that the shipped recipes' per-segment prefixes are pinned by a test."
  - "Per-card mechanical check: for each card with Rename or Delete groups, confirm the type-specific check ran against every target of the group."
  - "Test economy: check the new tests in the checklist, shuttle and shed packages against the covering tests that already exist, and against the budget in `cmd/lyx/testdata/test-budget.yaml`."
---

Round 1 focus, set from the plan's Shared Decisions and Card Index.
