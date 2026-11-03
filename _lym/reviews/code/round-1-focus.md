---
round: 1
focus:
  - "internal/heddleengine/shipped.json, lym-batching: compare each bullet against the old paragraph and settle whether any instruction was dropped, weakened or changed in meaning, notably the Read-tool-advice override, the reread-after-rewrite rule and the wait-then-send rule."
  - "internal/heddleengine/shipped.json, lym-plan-edits-before-gate: settle whether the four numbered steps and their sub-bullets keep the old meaning, notably that step 4 still says fix the report's findings by the same read-decide-edit order then run the gate again, and that the round definition and the three Never rules survive."
  - "Goal says the rule against reading one file in slices appears in both entries and is said once: settle whether exactly one entry now states it and the other loses no other instruction by it."
  - "Render path in internal/heddleengine/shipped.go and any renderer test: settle whether the entries' `##` headings follow `shippedHeading` with no level skip and whether the nested sub-bullets and the bullet list after the numbered list render as intended in the produced file."
  - "Goal asks that any stencil or doc quoting the old sentences be updated: settle whether another file under the diff's reach, such as internal/shuttleengine/claudeengine/batchingnudge.go or a stencil, quotes the old wording and needs a change."
  - "internal/heddleengine/store_test.go ShippedEntriesArePinned: settle whether the heading-only pin is enough for the Goal's requirement that tests follow the new text, and whether any other test pinning the shipped text or its render is left on the old sentences."
---

Diff is small: shipped.json, doc.go, store_test.go. The record carries only the Goal, so the Goal's Expected list is the measuring stick.
