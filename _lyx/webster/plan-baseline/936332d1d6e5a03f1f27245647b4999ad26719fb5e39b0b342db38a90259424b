# Card 7 — Detached pushes and board syncs exit under a pause, leaving their flags and commits for the next push

**Create:**
- `plan:internal/fabricengine#pushDeferral` -> `func pushDeferral(lockDir string, owed map[string]PushSpawn, pushed map[string]bool, exit func()) func()`
- `plan:internal/fabricengine#exitDeferred` -> `var exitDeferred = func() { os.Exit(0) }`
- `plan:internal/fabricengine#TestPushDeferral_PutsBackTheUnpushedFlags` -> `func TestPushDeferral_PutsBackTheUnpushedFlags(t *testing.T)`
- `plan:internal/fabricengine#TestDetachedPush_CoveredChildExitsBeforeItsFlags` -> `func TestDetachedPush_CoveredChildExitsBeforeItsFlags(t *testing.T)`
- `plan:internal/boardcli#TestSync_DeferrableExitsUnderACoveringPause` -> `func TestSync_DeferrableExitsUnderACoveringPause(t *testing.T)`
- `plan:internal/boardcli#exitDeferredSync` -> `var exitDeferredSync = func() { os.Exit(0) }`
- `plan:internal/fabricengine#TestCoalescePushRequest_CoveredCallDefers` -> `func TestCoalescePushRequest_CoveredCallDefers(t *testing.T)`

**Edit:**
- `internal/fabricengine#PushSpawn`
- `internal/fabricengine#pushSpawnCommand`
- `internal/fabricengine#statusPushRequest`
- `internal/fabricengine#markPending`
- `internal/fabricengine#foreignFlagRequest`
- `internal/fabricengine#DetachedPush`
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine#TestPushSpawnCommand`
- `internal/fabricengine_test#TestSpawnStatusPush`
- `internal/fabricengine#TestCoalesceFlagged_PendingFlags`
- `internal/boardcli/doc.go`
- `internal/fabriccli#addWeftVerbs`
- `internal/boardengine#spawnSync`
- `internal/boardcli#Command`
- `internal/fabricengine/doc.go`
- `internal/boardengine/doc.go`

**Uses:**
- `plan:internal/pauserecord#GateCovered`
- `plan:internal/pauserecord#SetGateDeferral`
- `plan:internal/fabricengine#ParentLockedFlag`
- `plan:internal/boardengine#DeferrableSyncFlag`
- `internal/pauserecord#InstallGate`
- `internal/parentlock#Store.HeldBy`
- `internal/parentlock#Dir`

**Intent:**
D6's child half.
A pause leaves the driver chain parked; only detached pushes and board syncs stop.
An exit under a pause pushes nothing more, leaves the commits local and the pending flags in place, writes one Info line and no failed-push record or creel mail, since the push is deferred, not failed (`PATTERN-push-failure-visible`).

Spawn-time parent lock.
`PushSpawn` gains `ParentLocked bool`.
`statusPushRequest`, for a task pair, sets it when `parentlock.Store{Dir: parentlock.Dir(BoardDir(l.HubPath))}.HeldBy(slug)` reports a live record naming the pair's slug, which holds exactly for the pushes a Finalize spawns past its parent-side merge; a failed read logs Warn and leaves it false.
`pushSpawnCommand` appends `--parent-locked` when set; `markPending` keeps it in the narrowed flag and `foreignFlagRequest` copies it, so a respawned flag keeps it.
`addWeftVerbs` registers the hidden persistent `ParentLockedFlag` bool beside `PauseSlugFlag`, and the bypass branch of `push` passes it into `DetachedPush`'s request.
`TestPushSpawnCommand` gains a `ParentLocked` row whose args carry `--parent-locked`; `TestSpawnStatusPush` gains a pair row with a live parent lock record of the slug (written through `parentlock.Store` with the test's own pid), whose request has `ParentLocked` set, a row whose parent lock directory path is taken by a plain file, whose request keeps it false and logs one Warn, and the existing rows keep it false.
`TestCoalesceFlagged_PendingFlags`'s respawn rows gain a foreign flag carrying `ParentLocked`, whose respawned request carries it, so `markPending` and `foreignFlagRequest` keep it.

Push child.
`DetachedPush`, when `req.ParentLocked` is false and `pauserecord.GateCovered()` reads covered, logs one Info line and returns nil before it takes the teardown lock or marks any flag; a parent-locked child never exits for coverage and keeps the gate's waiting behaviour against a hub entry (card 6).
`coalescePushSides` tracks the sides its step has pushed and installs `pauserecord.SetGateDeferral(pushDeferral(lockDir, owed, pushed, exitDeferred))` for the call's life; the deferral fires only in a process whose gate defers, which only the two marked children install, so `CoalescePushBothAt` in an ordinary process is unaffected.
`pushDeferral` returns the function that marks pending again every owed side not in `pushed`, logs one Info line naming the sides left local, and calls `exit`.
`TestPushDeferral_PutsBackTheUnpushedFlags` asserts, over a temp lock dir, that with `code` pushed and `records` not the records flag is on disk with its path and slugs, the code flag is not, and `exit` ran once; with neither pushed both flags are back, and with both pushed none is, `exit` running once in each row.
`TestDetachedPush_CoveredChildExitsBeforeItsFlags` installs a `Defer` gate over a temp record holding a live hub entry and calls `DetachedPush` with a teardown slug and worktree paths that do not exist: no teardown lock file is written, no failed-push record exists under `FailedPushDir(hub)` and the deferral's Info line is logged (`internal/testkit/logcapture`); with `ParentLocked` set the covered check is skipped, which the written teardown lock file shows.
`TestCoalescePushRequest_CoveredCallDefers` (integration tag, over a `gitkit` repository pair with a local bare origin) gives both sides an unpushed commit, installs a `Defer` gate whose record holds a live hub entry, swaps `exitDeferred` for a recorder that ends its goroutine with `runtime.Goexit`, runs `CoalescePushRequest` in that goroutine and asserts the recorder ran once at the first gated push, both pending flags are back on disk, origin holds neither commit and no failed-push record was written.

Board sync.
`spawnSync` appends `--deferrable` after `sync` and starts the child in the board directory (`cmd.Dir`), so its gate is held under the hub scope (card 6).
The board `sync` command in `boardcli.Command` registers the hidden `DeferrableSyncFlag` bool; with it set and `pauserecord.GateCovered()` reading covered it logs one Info line and succeeds with nothing committed or pushed, before `Sync` takes any lock, leaving the change for the next sync.
A deferrable sync that passed that check installs `pauserecord.SetGateDeferral` for its `Sync` call with a function that logs one Info line and calls `exitDeferredSync`, so a pause added before its push's network call makes it exit rather than park; its commit, if made, stays local for the next sync.
`TestSync_DeferrableExitsUnderACoveringPause` runs `sync --deferrable` under an installed `Defer` gate covered by a hub entry against a board path that does not exist and asserts a success envelope, then runs it without the flag and asserts the board error, which shows the deferred call touched nothing;
a third row runs `sync --deferrable` with the gate uncovered and asserts the same board error, which shows an uncovered deferrable sync proceeds to `Sync`.

The `fabricengine`, `boardengine` and `boardcli` docs state the deferral, the spawn-time parent-lock flag and its bound: it covers only pushes spawned while a parent lock record names their slug, and lets them pass only their own slug's owner entry.

**ImpactSummary:** one new PushSpawn field carried through the flag, respawn and spawn paths of fabricengine, one hidden flag each on fabric push and board sync; no exported signature changes.

**Commit:** 7: detached pushes and board syncs exit under a pause
