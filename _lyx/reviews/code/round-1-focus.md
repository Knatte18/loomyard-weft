---
round: 1
focus:
  - "internal/shellguard/rows.go and its tests: confirm the in-place-edit and redirect-write rows and only the helpers no other row uses are gone, and that the remaining rows (python, raw-go, refuse-raw, computed-rm, machine-sleep) are unchanged in pattern, steer and order."
  - "internal/shuttleengine/claudeengine settings.go, doc.go, settings_test.go and testdata/settings-golden.json: the golden diff is large, so settle whether it changes only what the two removed rows account for, with no other hook, notice sentence or ordering change under any pinned input combination."
  - "contracts/stencils/edit/edit-directive.md against the answer key: it must permit scripted edits, keep the lyx loom fmt route and the .scratch/ rule, and drop the Rename-card exception, without steering an agent to python or node. Settle whether the new wording contradicts any other stencil, spec or prompt text that still states the old ban or the rename_tools exception."
  - "Stale descriptions of the removed rows or the ban across the repo (docs, doc.go files, PATTERN and pattern/ background files, the no-denied-recovery scan rows in cmd/lyx/norecoverydeny_test.go, stencils.go comments, editdirective tests): settle whether any still names sed -i, perl -i or the redirect rule as denied."
  - "Test economy in the shellguard, editdirective and claudeengine test diffs: settle whether any new or rewritten test duplicates existing coverage or leaves a behavior the diff adds uncovered."
---

Seed focus from the board entry, the decision record and the diff stat; no round has been judged yet.
