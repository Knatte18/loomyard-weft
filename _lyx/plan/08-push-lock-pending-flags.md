# Card 8 — push-lock-pending-flags

**Create:**
- `plan:internal/fabricengine#pendingFlagPath` -> `func pendingFlagPath(lockDir, side string) string`
- `plan:internal/fabricengine#markPending` -> `func markPending(lockDir string, owed map[string]PushSpawn) error`
- `plan:internal/fabricengine#takePending` -> `func takePending(lockDir, side string) (req PushSpawn, found bool, err error)`
- `plan:internal/fabricengine#coalesceFlagged` -> `func coalesceFlagged(lockDir string, owed map[string]PushSpawn, acquire func(lockPath string) (*lock.FileLock, bool, error), step func() (progressed bool, err error)) (acquired bool, err error)`
- `plan:internal/fabricengine#CoalescePushRequest` -> `func CoalescePushRequest(req PushSpawn, opts SyncOptions) (res PushResult, err error)`
- `plan:internal/fabricengine#TestCoalesceFlagged_PendingFlags` -> `func TestCoalesceFlagged_PendingFlags(t *testing.T)`

**Edit:**
- `internal/fabricengine#acquirePushLock` -> `func acquirePushLock(lockWait time.Duration) func(lockPath string) (*lock.FileLock, bool, error)`
- `internal/fabricengine#CoalescePushBothAt`
- `internal/fabricengine#PushAnchored`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabriccli#addWeftVerbs`
- `internal/fabricengine_test#TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#PushSpawn`
- `plan:internal/fabricengine#statusPushRequest`
- `internal/fabricengine#coalescePush`
- `internal/fabricengine#spawnDetachedPushFn`
- `internal/fabricengine#weftPushLockFile`
- `internal/fabricengine#ensureWeftLockDirAt`
- `internal/lock#TryAcquireWriteLock`
- `internal/lock#AcquireWriteLockWithin`
- `internal/lock#AcquireWriteLock`

**Intent:** Change only the acquisition of fabric's records push lock (`fabric.push.lock` under a records worktree's `.weft/` lock directory) so a child that finds it held exits, and the live pushers per records worktree are bounded to the holder, with no committed transition left unpushed (D1's pending-flag protocol).
`Bolt.Sync`'s blocking coalescing over `board.push.lock` keeps `coalescePush`, which this card leaves untouched.

A pending flag is one file per side, `pending.records` or `pending.code`, beside the push lock in the same lock directory, which `seedWeftArtifactExcludes`' `.weft/` entry already keeps out of git.
Its content is the JSON of the single-side `PushSpawn` that owes it: that side's path and the pause and teardown slugs of the push it serves, so any holder can respawn it.
`markPending` writes one flag per owed side, `takePending` reads and removes one, and `pendingFlagPath` names it.

`coalesceFlagged(lockDir, owed, acquire, step)` is the holder loop every pusher of `fabric.push.lock` runs:
1. `markPending(lockDir, owed)`, so a commit always precedes its flag;
2. `acquire` the lock; not acquired returns `(false, nil)`, leaving the flags and the commits to the holder;
3. under the lock, take every owed side's flag before each `step`, and loop while `step` reports progress; a step error releases and returns it;
4. after the last no-progress step, release the lock and re-read every flag: a flag of an owed side set again goes back to step 2 without re-marking, and when only a flag of a side the holder does not push is set, the holder takes it and spawns that side's push through `spawnDetachedPushFn` with the request the flag holds;
5. with no flag left, return `(true, nil)`.
Since a commit precedes its flag and every holder re-reads the flags after it releases, the last holder to release always sees a flag set after its last push, and no committed transition is left unpushed.

`CoalescePushRequest(req, opts)` is the detached child's push: its owed sides are `code` when `req.WarpPath` is set and `records` when `req.WeftPath` is, each owed request being `req` narrowed to that side; its `acquire` is `TryAcquireWriteLock`, so a child that finds the lock held exits at once instead of queueing behind the holder however long it hangs; its step is `CoalescePushBothAt`'s per-side step, which moves into a shared helper.
`CoalescePushBothAt` keeps its signature and its blocking acquire for its foreground callers (hubforge's fixtures and tests), running the same step under `coalesceFlagged` with owed requests carrying no slugs.
`acquirePushLock(lockWait)` returns the synchronous pushers' acquire: `LockWaitUnbounded` blocks as today, and a positive wait gives up through `AcquireWriteLockWithin`.
`PushAnchored` (owing `records`) and `PushPairAnchored` (owing `records`, and `code` outside the prime) run their push as `coalesceFlagged`'s step with that acquire, their owed requests from `statusPushRequest(l, "")`; a synchronous pusher that did not acquire within its wait returns the `ErrPushLockBusy` error it returns today, with nothing pushed.
So a code commit made while `commit-records` holds the lock in a pair is pushed once it releases: the code side's child marks `code`, finds the lock held and exits, and `PushPairAnchored`, which owes `code`, re-reads the flag after releasing and loops.
`fabriccli`'s push bypass builds the child's `PushSpawn` from `--warp-path`, `--weft-path` and the two slug flags and calls `CoalescePushRequest`.

Tests:
- `TestCoalesceFlagged_PendingFlags`, Tier 1 over a temp lock directory and a step closure, no git, rows: with the lock held by another `FileLock` and a try-acquire, the call returns `(false, nil)` without running the step and leaves `pending.records`; a step that, on its first run, marks `records` pending as a second pusher would makes the holder run the step again after releasing, and the call ends with no flag left; a holder owing `records` that finds a `pending.code` flag after releasing spawns exactly that flag's request through a swapped `spawnDetachedPushFn` and runs no further step; and a step returning an error makes the call return that error with the lock released, a second try-acquire then succeeding.
- `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` gains a row: with the push lock held, `CoalescePushRequest` returns nil at once with nothing pushed and a `pending.records` flag on disk.
The existing `TestCoalescePushBothAt_*` and `TestPushAnchored_*` tests pin the foreground and synchronous pushes unchanged.

`internal/fabricengine/doc.go` states the pending-flag protocol and that it covers `fabric.push.lock` alone.

**ImpactSummary:** The three pushers of `fabric.push.lock` share one flagged loop; synchronous and foreground callers keep their signatures, wait semantics and errors, and only the detached child stops queueing; Bolt's board push lock is untouched.

**Commit:** 8: fabricengine: push lock try-lock with per-side pending flags
