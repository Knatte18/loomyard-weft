---
round: 1
focus:
  - "Amendment scope: read `git diff b9597b2e1a9a4f722b923e8aa95a4b8b5aacb00a main -- docs/overview.md internal/landingshed/doc.go` against the task's own edits to the same two files; settle whether the merged result keeps the task's changes (the `recipe-owns-failures` section in `internal/landingshed/doc.go`, regrouped overview references) and whether any parent addition cites a removed `PATTERN-` ID."
  - "Settle whether any `PATTERN-` ID the parent's new or changed text names (in `docs/overview.md`, `internal/landingshed/doc.go`, and the files the parent's recent commits touched, such as gate test tiering) is one the decision record removes, and so needs the sweep the record asks for."
  - "In `PATTERN.md`, check that each of the record's `PATTERN changes` entries landed with its declared text, and that no parent-added entry was lost or left unlinked by the merge; confirm `pattern.Check` expectations (one background file per linked entry, no orphan) still hold after the merge."
  - "In `internal/landingshed/doc.go`, check the merged doc comment keeps one sentence per line and states the `recipe-owns-failures` rule with its bounds and enforcement, without duplicating or contradicting the parent's edit."
---

Seed focus for an amendment round: the parent's merge touched two files the task also changed, so round 1 starts there.
