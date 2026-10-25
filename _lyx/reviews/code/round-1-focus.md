---
round: 1
focus:
  - "docs/overview.md after the Parent-Align merge: do the orch, batten and beam module rows still carry the beam-window wording (window named `beam: prime` / `beam: <slug>`, dobby section first, each run shown once, orch lines of the own orch and every beam below) and none of the old 'overview window' wording the parent's edits could have reintroduced? Read `git diff 1c344094affd7a89153401258ff15083669e05d4 main -- docs/overview.md` against the branch's version of the same rows."
  - "Parent changes the amendment names, such as the shortcut set in the shortcutcli row and the removed `lyx start` alias line: does the merged file keep the parent's text where the task did not touch it, with no hunk of the task's change lost or duplicated by the merge?"
  - "Whether code or doc comments elsewhere in the branch's diff (battencli, beamcli, orchcli, reedengine) still name the overview window or a window order that the parent's changes to the shortcuts or window set make stale."
---

Amendment round: the settled remainder is out of scope; the subject is the Parent-Align result for `docs/overview.md`, the one file both sides changed.
