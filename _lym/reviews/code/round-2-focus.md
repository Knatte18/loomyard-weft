---
round: 2
focus:
  - "internal/shuttleengine/reinterpret.go `DropEditLoopEdits`: for each value of `edit_loop_edits` (0, null, empty, non-zero, non-scalar), does the result keep what the hub meant, per PATTERN-config-layers (rewrite to the old meaning, or flag with a way forward), and does the flagged line name a way forward the shell and settings allow?"
  - "internal/shuttleengine/reinterpret_test.go `TestDropEditLoopEdits`: does each row assert which of the rewrote and flagged lists holds the line, and do the rows cover a null value and a document whose top level is not a mapping?"
  - "internal/shuttleengine/editloop.go `fold` and `TestWait_EditLoopNotice` in editloop_test.go: does some row fail when the pruning of the repeat history against the one-minute window is removed?"
---

The focus targets the new text the first round's fixes add; settled code is out of scope.
