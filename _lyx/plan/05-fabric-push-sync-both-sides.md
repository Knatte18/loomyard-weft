# Card 5 — lyx fabric push and sync push both sides

**Edit:**
- `internal/fabriccli/weft_verbs.go`
- `internal/fabriccli/spawn.go`
- `internal/fabriccli/hubmutation_integration_test.go`
- `internal/fabriccli/weftskippush_integration_test.go`
- `internal/fabricengine/coalesce_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#PushPairAnchored`
- `plan:internal/fabricengine#LockWaitUnbounded`
- `internal/fabriccli#addWeftVerbs`
- `internal/fabriccli#spawnPush`
- `internal/fabricengine#SpawnDetachedPush`

**Intent:**
Decision 5 of the decision record.

`push`'s normal mode replaces `fab.PushWeft(opts)` with `fabricengine.PushPairAnchored(l, opts, fabricengine.LockWaitUnbounded)`, extending the verb's record with the result's mutations as today.
It waits out a detached child holding the lock, as the repo-root lock it replaces waited, and a rejection that stays after the one retry is an error envelope naming the side, never a swallowed warning.
It no longer runs `pull --rebase` on a rejection.
The bypass branch keeps `CoalescePushBothAt`.

`sync` hands the detached child both paths: `spawnPush` takes the warp worktree path beside the weft worktree path and passes both to `SpawnDetachedPush`, and `sync` passes `l.WorktreePath()` and `WeftWorktree(l)`.
The verb's record holds one `KindPushSpawned` entry per side handed to the child.

What this lets through, and its bound: both verbs push both sides unconditionally, so in the prime they publish unpushed commits of the parent branch even when the call committed nothing, including the `sync` that `lyx config` runs after an edit.
The push is a plain, rebase-free push of commits already on the checked-out branch, never a force, and a branch that is behind, diverged or protected on its remote is rejected and reported, with nothing rewritten.

The `push` and `sync` `Long` help, the `Short` texts, the file header of `weft_verbs.go` and `spawn.go`'s comments say both sides are pushed, and that `push` reports a rejection as an error after pushing the other side.
`coalesce_integration_test.go`'s comment calling the warp-less call the `sync` production path is corrected, since `sync` now hands both paths.

Tests:

- `hubmutation_integration_test.go`'s `TestRunCLI_HubMutationScenario` gains two steps:
  after commits on both sides of the prime, `lyx fabric push` exits 0 and both bare remotes sit at their local HEADs;
  placed with the late, damaging steps, with the prime's code branch diverged on its bare by a second clone, `lyx fabric push` returns an error envelope naming the warp side and its worktree, the weft bare sits at the weft HEAD, and the local warp HEAD is unchanged.
- `weftskippush_integration_test.go`'s sync step asserts the record holds a `push_spawned` entry for each side, and its push step asserts neither bare moved under `WEFT_SKIP_PUSH`.

**ImpactSummary:** `PushPairAnchored` gains its second production caller (the `push` verb) and `spawnPush` keeps its one caller (`sync`); `Fabric.PushWeft` loses its only production caller and keeps a test caller only; the detached child's bypass path is unchanged.

**Commit:** 5: fabric: push and sync push both sides
