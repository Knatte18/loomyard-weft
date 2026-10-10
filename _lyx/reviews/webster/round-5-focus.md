---
round: 5
focus:
  - "PATTERN.md size: confirm `wc -c PATTERN.md` is at most 16384 bytes (`MaxOverviewBytes` in `internal/pattern/check.go`), that `TestOverview_RealFilePassesCheck` passes, and that the `PATTERN-hub-store-gc` line still names the in-flight skip and no other entry lost its constraint."
  - "`internal/hubreconcile/walk.go` `reconcileHubWide`: settle what happens for a board with no upstream, given `PullThenCommitWritten` lets such a board write while `PushCoalesced` fails on it, and whether the build stamp can still be written."
  - "`internal/fabriccli/fabric.go` reconcile write: settle whether paths of pruned legacy config files reach the `file_written` mutations that `Bolt.PullThenCommitWritten` records."
  - "`internal/fabricengine/stencilcommit.go` and `junctionnames.go`: settle whether the file header and the `BoardWriteLockFile`, `StencilsSubtreeRel` and `SpecsSubtreeRel` comments describe the seeding commit path as it now stands."
---

Rationale: round 4's only blocking item is the size cap, so the first entry verifies the fix; the others check the remaining open findings.
