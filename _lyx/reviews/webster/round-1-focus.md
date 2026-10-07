---
round: 1
focus:
  - "Cards 6, 7 and 10: `commentlint` and `lyx loom lint-comments` must flag only the comment breaks a diff creates; settle whether a wrap in untouched comment lines, or a legitimate semantic break, is wrongly flagged or missed."
  - "Card 8 and card 10: `impactset` must fall back to the full plan verify wherever it cannot tell (unreadable record, missing base commit, non-Go or config change, `//lyx:guard` tests); settle whether any such path instead yields an empty or too-narrow command."
  - "Card 1: `verifytree` per-command records and `LatestPass` for the round gate's base; settle whether a record for one command can be mistaken for another's, or a stale or failed pass be taken as the base."
  - "Cards 2 to 5: after the `planindex` seam, settle whether `websterengine` and `loomshed` still import `planglyph` or the code index directly or transitively, and whether the tree-sitter link guard's allowed list matches the packages that really need it."
  - "Card 12: `publish_verify` config loading and its run after the plan verify in Publish; settle whether an absent key, a failing command and the old `landing.yaml` template behave as the plan's Shared Decisions state."
  - "Card 14: the test budget guard's per-package counting and its pinned after-state; settle whether the count is derived rather than hand-copied and matches the repository's tests."
  - "Cards 9 and 14, test economy: settle whether any added test duplicates an existing test's coverage, tests a helper the public surface already covers, or leaves a behavior this diff adds uncovered."
  - "Per-card mechanical check: for each card's groups, confirm the type-specific check (Rename script plus grep, Delete `assert-no-callers`) ran against every group's own targets."
---

Seed focus from the plan's card index and shared decisions; no review round has run yet.
Cards 11, 13 and 15 are mostly config, docs and measurement, so they carry no focus entry beyond comment-convention compliance, which the base review covers.
