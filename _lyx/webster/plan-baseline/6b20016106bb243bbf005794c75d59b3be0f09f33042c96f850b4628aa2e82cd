# Card 4 — ensure-leaves-machine-file

**Edit:**
- `internal/hubreconcile_test#runStaleBuildReconcile`

**Uses:**
- `internal/configengine#MachineFile`

**Intent:**
Pin decision record D5 for the hub reconcile after a binary change: `hubreconcile.Ensure` never reads, rewrites or removes a machine file.
`runStaleBuildReconcile` writes `slots: 1\ngo_parallel: 8\n` to `configengine.MachineFile(<board dir>, "gate")` before it runs `Ensure`, and after the walk asserts the file's bytes are unchanged; the board's committed files and the commit counts the function already asserts stay as they are, since the file sits under the board dir's never-tracked `.lyx/`.
No production code changes: `Ensure` writes only `_lyx/config/` files, the build stamp and its lock.

**ImpactSummary:** Test-only, one helper of `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` in the integration tier; its three rows each gain the assertion.
