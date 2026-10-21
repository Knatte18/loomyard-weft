---
round: 2
focus:
  - "PATTERN.md overview lines for PATTERN-hub-wide-config, PATTERN-hub-store-gc and PATTERN-friction-capture: compare each against the parent's text at 860c5d0334a144bf75c5d423272b4ec681493772 and settle whether the parent's wording (machine override beside the file, no automatic gc or maintenance, note reflected at the run's end and archived once covered) survives in the merged overview, and whether any bytes cut to keep headroom came from an entry neither side changed in this merge."
  - "PATTERN.md line for PATTERN-origin-reconcile: settle whether it matches the declared text exactly, ending in 'the sole judge of local branches against origin', and whether the overview still passes the headroom check in internal/patternlint."
  - "internal/fabricengine/remove_localmode_integration_test.go, TestRemoveLocal and the TestSnapshotUncommitted case asserting exact snapshot paths: settle whether a populated, linked root .scratch under a pair is asserted to be absent from the snapshot paths and the force dirty gate, and whether the RemoveLocal record holds one link_removed at the root .scratch ahead of the warp worktree_removed."
---

Round 1 findings were fixed in commits after the merge.
Round 2 reviews that new text as it stands at HEAD.
