---
round: 2
focus:
  - "In internal/hubreconcile/walk.go, `reconcileWorktree` and `reconcileHubWide` write config files before they commit. Settle whether every failure after the first write (a `configsync.ReconcileAll` error, a non-merge `Commit` error, a hub-wide write error) leaves the worktree or board directory exactly as it was before the call, so a retry cannot see the written files as current and stamp the build over them."
  - "In internal/preflightshed/preflight.go, `reconcileRefusal` maps a `*hubreconcile.WorktreeError` (and any other error) to a refusal message. Settle whether each remaining branch is reachable from the inputs `Ensure` can produce, whether each is reached by a test, and whether every message names the way forward."
  - "In internal/loomcli/arm.go, `loom start` and `loom resume` call `hubreconcile.Ensure` before arming. Settle whether a reconcile failure reaches the operator as a non-zero exit through the built binary, with the file named and the way forward stated, and whether a test pins it."
  - "In internal/hubreconcile/walk.go, `snapshotConfig` reads a config file for later restore. Settle whether every case it handles can occur given what `os.ReadFile` returns."
---

Focus targets the code a fresh reviewer has not seen: the fix commits after the first review.
