```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that each refusal, next step, row or test the fix added can be reached and leads to progress.

**Findings**

1. **MEDIUM.** Changed text: the new D9 bullet "A call naming no package and lacking `--upkeep` (D8) runs as a pass but leaves the exit record as it was". It conflicts with D9's Bound, "on a hub with fewer than two slots only hand calls build maps".
   - Summary: on a hub with fewer than two slots, a bare `lym gate testmap` is a hand call that builds nothing. The Bound's way forward works only for a hand call that names packages.
   - Evidence: D9 says "On a hub with fewer than two slots a pass takes none: it plans, re-stamps and ends." The new bullet makes the bare call a pass, and the verb's examples document the bare form (review F8). An operator who follows the Bound with the documented bare call ends with nothing built. The Bound or the bullet needs to say that only a call naming packages builds on such a hub.

2. **MEDIUM.** Changed text: the new D8 bullet "A hand caller may pass it too: … The build lock per package keeps it and any other pass off the same package, and the dobby starts no pass while a pass registered as `runproc.RoleTestMap` lives."
   - Summary: a hand-run `--upkeep` call is not registered as `runproc.RoleTestMap`. The dobby can therefore start a second pass beside it, and both write the exit record. The stated bound does not cover that case.
   - Evidence: the doc comment of `startTestMap` at `internal/beamcli/dobbydeps.go:591-598` says the dobby registers the process it starts: "It registers the process under the hub owner in runProcDir as the test-map refresh". The verb does not register itself. The record never says that a `--upkeep` call registers itself, and D8's own Bound already admits "a pass started beside an unregistered one wastes work". The bullet needs to say either that the verb registers itself under `--upkeep`, or that a hand `--upkeep` call is unregistered and its exit record can race the dobby's pass.

Two other ways forward hold. The new D8 criterion, that a prime `_lym/config/beam.yaml` or local file holding `test_map_every: 0` reads as `test_map_upkeep: false`, can be reached: `hubreconcile`'s walk calls `configsync.ReconcileAll` with `Migrate: true` (`walk.go:341`), and `reconcileSparseFile` runs `m.Migrate` before `pruneSparse` on both the worktree layer and the local layer. The D9 yield rule leaves a hand call no deadlock: the hand call never waits on the pass's slot, so a pass that waits for the lock while holding a slot only waits for one job.

Tool calls:
- Read `configsync.go:505-540` and `:659-690` and grepped the `ReconcileAll` callers. My context showed only which functions call `Migrate`, not whether the worktree and local layers are migrated before the prune or which walk runs them.
