---
round: 2
focus:
  - "internal/driftengine/verbs.go RecordScope: settle whether a capture taken under partial skipping a test, followed by a replacement to scope off and then a replacement to full, can leave a capture that lacks that test's profile counted under the full scope."
  - "internal/driftengine/verbs.go RecordScope: settle whether a failed removal of the capture record can leave the replacement scope stored so that a retry no longer sees the skip difference."
  - "internal/driftengine/baseline.go captureAgain: settle which StartCapture errors become the capture halt offering lym drift defer, and whether an environment fault such as a failed binary lookup, an uncreatable child log or a held capture lock is among them."
  - "internal/loomcli/driftwiring.go and internal/driftcli: settle whether a test with a subdirectory anchor (fixture AnchorRel not \".\") fails when a call site hands the engine the worktree root instead of the anchored directory, across the capture child spawn, the compare and side-capture commands, and driftDeps."
  - "internal/driftengine/steprun.go acquireSlot against internal/gateslot Holder.Worktree: settle whether the holder identity and its doc comment agree now that the anchored directory is recorded."
---

Focus targets the sites where round 1 found defects, so a fresh reviewer re-checks them without a prior verdict.
