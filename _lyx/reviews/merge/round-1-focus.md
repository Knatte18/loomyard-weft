---
round: 1
focus:
  - "internal/battencli/wire_seams_test.go: the parent dropped the PrepareDiscussionSpawn and PreparePlanSpawn entries from intentionallyNil, the task changed pauserecord.Network's call shape in TestWire_PointsTheNetworkGateAtItsSlug; settle whether the merged test file still matches the merged Wire seams and the merged pauserecord.Network signature."
  - "docs/overview.md: the parent reworded the twill producer paragraphs of the Discussion and Planner entries, the task renamed pause to tender in the module table, the shortcuts bullet and the module bullet; settle whether both sets of edits read consistently in the merged text and name only symbols and packages that exist after the merge."
  - "Callers of pauserecord.Network and of the renamed tendercli and tenderengine packages in code the parent touched (internal/loomcli, internal/loomengine, internal/shedrecipe, internal/testkit/envkit): settle whether any still uses the old package names or the old single-return Network form."
  - "internal/shortcutcli/shortcuts.yaml against the parent's new loomcli and loomengine wiring: settle whether the task's pause and resume shortcut entries collide with, or are assumed absent by, anything the parent added."
---

Seed focus from the two overlapping files and the rename and signature changes that cross the parent's touched packages.
