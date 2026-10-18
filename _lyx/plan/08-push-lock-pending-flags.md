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
- `internal/fabricengine#RecordsLockDirPath`
- `internal/fabricengine#BoardDir`
- `internal/weftname#SiblingPath`
- `internal/fsx#AtomicWriteBytes`
- `internal/lock#TryAcquireWriteLock`
- `internal/lock#AcquireWriteLockWithin`
- `internal/lock#AcquireWriteLock`

**Intent:** Change only the acquisition of fabric's records push lock (`fabric.push.lock` under a records worktree's `.weft/` lock directory) so a child that finds it held exits, and the live pushers per records worktree are bounded to the holder, with no committed transition left unpushed (D1's pending-flag protocol).
`Bolt.Sync`'s blocking coalescing over `board.push.lock` keeps `coalescePush`, which this card leaves untouched.

A pending flag is one file per side, `pending.records` or `pending.code`, beside the push lock in the same lock directory, which `seedWeftArtifactExcludes`' `.weft/` entry already keeps out of git.
Its content is the JSON of the single-side `PushSpawn` that owes it, holding that side's path and the pause and teardown slugs of the push it serves and nothing else, so any holder can respawn it.
A holder's respawn sets `WorkDir` itself to `BoardDir` of the hub, the parent of the records worktree the lock directory belongs to, and ignores any stored value, so a flag written by a foreground pusher, which carries no `WorkDir`, never starts a child in the holder's own directory, possibly a pair worktree.
`markPending` writes one flag per owed side through `fsx.AtomicWriteBytes`, so a reader sees either no flag or a whole one, `takePending` takes one by renaming it to a name private to the taking process in the same lock directory, then reading and removing that file, so no process ever holds a file at a flag path open and `markPending`'s rename onto it never meets a Windows sharing violation; a flag whose content does not decode is still removed and reported found with an error naming the flag that wraps the JSON decode error, so its caller tells it from a rename, read or remove failure; and `pendingFlagPath` names it.

`coalesceFlagged(lockDir, owed, acquire, step)` is the holder loop every pusher of `fabric.push.lock` runs:
1. `markPending(lockDir, owed)`, so a commit always precedes its flag;
2. `acquire` the lock; not acquired leaves the flags and the commits to the holder and returns `(false, nil)` on the first acquire, and `(true, nil)` on a re-acquire after a step already ran;
3. under the lock, take every owed side's flag before each `step`, and loop while `step` reports progress; a step error releases and returns it;
4. after the last no-progress step, release the lock and re-read every flag: a flag of an owed side set again goes back to step 2 without re-marking, and when only a flag of a side the holder does not push is set, the holder takes it and spawns that side's push through `spawnDetachedPushFn` with a request built from the flag's validated side path and slugs only, never copying its opposite-side path field (a `pending.code` flag's request keeps only `WarpPath`, a `pending.records` flag's only `WeftPath`, the other left empty), its `WorkDir` set to the board directory;
5. with no flag left, return `(true, nil)`.
Since a commit precedes its flag and every holder re-reads the flags after it releases, the last holder to release always sees a flag set after its last push, and no committed transition is left unpushed.

`CoalescePushRequest(req, opts)` is the detached child's push: its owed sides are `code` when `req.WarpPath` is set and `records` when `req.WeftPath` is, each owed request being `req` narrowed to that side; its `acquire` is `TryAcquireWriteLock`, so a child that finds the lock held exits at once instead of queueing behind the holder however long it hangs; its step is `CoalescePushBothAt`'s per-side step, which moves into a shared helper.
`CoalescePushBothAt` keeps its signature and its blocking acquire for its foreground callers (hubforge's fixtures and tests), running the same step under `coalesceFlagged` with owed requests carrying no slugs.
`acquirePushLock(lockWait)` returns the synchronous pushers' acquire: `LockWaitUnbounded` blocks as today, and a positive wait gives up through `AcquireWriteLockWithin`.
`PushAnchored` (owing `records`) and `PushPairAnchored` (owing `records`, and `code` outside the prime) run their push as `coalesceFlagged`'s step with that acquire, their owed requests from `statusPushRequest(l, "")`; a synchronous pusher returns the `ErrPushLockBusy` error it returns today, with nothing pushed, only when its first acquire misses its wait; a missed re-acquire after a completed step leaves the flag to the current holder and returns nil, since its own push already landed, which follows from `coalesceFlagged` reporting `acquired` true once a step ran.
So a code commit made while `commit-records` holds the lock in a pair is pushed once it releases: the code side's child marks `code`, finds the lock held and exits, and `PushPairAnchored`, which owes `code`, re-reads the flag after releasing and loops.
`fabriccli`'s push bypass builds the child's `PushSpawn` from `--warp-path`, `--weft-path` and the two slug flags and calls `CoalescePushRequest`.

What this lets through: a child that finds the lock held exits and leaves its commits to the holder, so a holder that hangs delays every later push of that records worktree until it exits; and a holder respawns a flag's side path and its pause and teardown slugs, with `WorkDir` set to the board directory.
What bounds it: flags live only in the records worktree's untracked `.weft/` lock directory, and only `markPending` writes them; a holder respawns a flag of a side it does not push (step 4) only when that side's path is the worktree the lock directory belongs to, for `records` a `WeftPath` whose `RecordsLockDirPath` is the lock directory, and for `code` a `WarpPath` whose records sibling (`weftname.SiblingPath` of its parent and base name) has that lock directory, and drops any other such flag with a Warn, spawning nothing for it; the respawned request carries only that side's path, never the flag's opposite-side path field; a flag whose `PauseSlug` or `TeardownSlug` holds a path separator or a `..` element is dropped with a Warn naming the flag, spawning nothing, since `TeardownLockPath` builds a file path from the teardown slug; a flag of a side the holder does not push that does not decode as JSON is dropped with a Warn naming the flag, spawning nothing, and for one of an owed side the holder ignores `takePending`'s decode error and treats the flag as set, as it would a whole one, so a corrupt flag never makes a holder return an error; a flag of an owed side is only taken, never respawned, since the holder pushes that side from its own request; the hung-holder residual is the one D1 accepts.

Tests:
- `TestCoalesceFlagged_PendingFlags`, Tier 1 over a temp lock directory and a step closure, no git, rows: with the lock held by another `FileLock` and a try-acquire, the call returns `(false, nil)` without running the step and leaves `pending.records`; a step that, on its first run, marks `records` pending as a second pusher would makes the holder run the step again after releasing, and the call ends with no flag left; one table-driven group of respawn rows, generated over both flag sides and every field the respawn reads, with a swapped `spawnDetachedPushFn` recording each spawn, where the two sides are a holder owing `records` that finds `pending.code` after releasing (side path `WarpPath`, which must be the lock directory's paired code worktree, opposite-side path `WeftPath`) and a holder owing only `code` under the paired code worktree, as `CoalescePushRequest` builds for a code-only child, that finds `pending.records` (side path `WeftPath`, whose `RecordsLockDirPath` must be the lock directory, opposite-side path `WarpPath`), and each side has these rows: (a) a valid flag respawns exactly one request carrying only that side's path, its two slugs and `WorkDir` equal to the board directory even when the flag file holds another one, and runs no further step; (b) a side path that is not the lock directory's own worktree is removed with a Warn naming the path and spawns nothing; (c) a flag that also carries the opposite-side path respawns a request whose opposite-side path is empty; (d) a `PauseSlug` that is a `..` element, and one holding a path separator such as `a/b`, each remove the flag with a Warn naming it and spawn nothing; (e) the same two cases for `TeardownSlug`; (f) a flag file that does not decode as JSON is removed with a Warn naming it and spawns nothing; rows (b) to (f) take the valid flag of (a) and change only the one field or the file's bytes, so none can pass on another bound; a holder owing `records` whose step writes non-JSON bytes to `pending.records` on its first run, after `markPending` has run, takes that flag after releasing with no Warn and no error, runs the step once more and returns `(true, nil)` with no spawn and no flag left; a holder owing `code` under a code path that is not that paired worktree, as `CoalescePushBothAt`'s foreground callers pass, takes its own `pending.code` flag with no Warn; a step that marks `records` pending again, with an `acquire` that grants the first call and refuses the second, returns `(true, nil)` and leaves that flag on disk, the case where a synchronous pusher returns nil rather than `ErrPushLockBusy`; and a step returning an error makes the call return that error with the lock released, a second try-acquire then succeeding.
- `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` gains a row: with the push lock held, `CoalescePushRequest` returns nil at once with nothing pushed and a `pending.records` flag on disk.
The existing `TestCoalescePushBothAt_*` and `TestPushAnchored_*` tests pin the foreground and synchronous pushes unchanged.

`internal/fabricengine/doc.go` states the pending-flag protocol and that it covers `fabric.push.lock` alone.

**ImpactSummary:** The three pushers of `fabric.push.lock` share one flagged loop; synchronous and foreground callers keep their signatures and wait semantics, and their errors but for a missed re-acquire after a landed push, which now returns nil; only the detached child stops queueing; Bolt's board push lock is untouched.

**Commit:** 8: fabricengine: push lock try-lock with per-side pending flags
