# Card 3 — fabricengine pushes both sides under the absorbing lock, retrying once

**Create:**
- `plan:internal/fabricengine#PushPairAnchored` -> `func PushPairAnchored(l *lyxcwd.Location, opts SyncOptions, lockWait time.Duration) (res PushResult, err error)`
- `plan:internal/fabricengine#IsPushRejected` -> `func IsPushRejected(err error) bool`
- `plan:internal/fabricengine#ErrPushLockBusy` -> `var ErrPushLockBusy = errors.New("fabricengine: push lock busy")`
- `plan:internal/fabricengine#StatusPushLockWait` -> `const StatusPushLockWait = 30 * time.Second`
- `plan:internal/fabricengine#LockWaitUnbounded` -> `const LockWaitUnbounded time.Duration = 0`

**Edit:**
- `internal/fabricengine/pushanchored.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/export_integration_test.go`
- `internal/fabricengine/pushanchored_integration_test.go`
- `internal/battencli/commitstatus.go`
- `internal/battencli/wire.go`
- `internal/loomcli/wiring.go`

**Uses:**
- `plan:internal/lock#AcquireWriteLockWithin`
- `plan:internal/gitrepo#Repo.HasUnpulled`
- `internal/lock#AcquireWriteLock`
- `internal/gitrepo#Repo.PushRebaseFree`
- `internal/gitrepo#Repo.Fetch`
- `internal/fabricengine#ensureWeftLockDirAt`
- `internal/fabricengine#weftPushLockFile`
- `internal/fabricengine#recordPushIfAdvanced`
- `internal/fabricengine#CoalescePushBothAt`
- `internal/fabricengine#PushResult`
- `internal/fabricengine#SyncOptions`

**Intent:**
Decisions 1, 3 and 4 of the decision record, all inside `pushanchored.go`.

Two unexported helpers carry the shared mechanics:

- The lock helper takes the weft worktree path and `lockWait`, ensures the lock directory through `ensureWeftLockDirAt`, and acquires `fabric.push.lock` under it, the same file `CoalescePushBothAt` takes.
  `LockWaitUnbounded` (zero) blocks through `lock.AcquireWriteLock`, as the detached child does; any positive wait goes through `lock.AcquireWriteLockWithin`.
  An expiry returns an error wrapping `ErrPushLockBusy` with `%w`, naming the lock path and the wait and stating that neither side was pushed; it names no side.
- The retry helper runs `PushRebaseFree` on one repo; on `gitrepo.ErrPushRejected` it runs `Fetch`, then `HasUnpulled`, and pushes once more only when the fetch succeeded and `HasUnpulled` reports false, meaning the remote tip is already contained in local HEAD.
  Otherwise, or when the second push is rejected again, it returns the bare `gitrepo.ErrPushRejected`.
  Bound: one fetch and one extra push per side per call, never a rebase, merge or force; a remote holding commits the local branch lacks is left alone.
  It never touches `gitrepo.PushRebaseFree` itself, whose other callers keep its plain contract.

`PushPairAnchored(l, opts, lockWait)` is the new vocabulary-neutral entry.
`opts.SkipGit` or `opts.SkipPush` returns an empty result before any lock is taken.
Otherwise it takes the lock, then pushes the code side (`l.WorktreePath()`) and then the records side (`WeftWorktree(l)`) through the retry helper, attempting the records side even when the code side failed.
A side whose HEAD is unborn is skipped, as `CoalescePushBothAt` skips it.
Each side's failure is wrapped as `fabricengine: push <side> side at <path>: %w`, with `warp` and `weft` as the side names fabric owns, so `errors.Is(err, gitrepo.ErrPushRejected)` holds for a rejected side; both failures return `errors.Join` of the two.
Each side that observably advanced is recorded through `recordPushIfAdvanced` into a `Mutations` rooted at `l.HubPath`, embedded in the returned `PushResult`.

`IsPushRejected(err)` is `errors.Is(err, gitrepo.ErrPushRejected)`, the side-neutral predicate `loomcli` reads without importing `gitrepo`.
`StatusPushLockWait` is the bounded wait the per-transition status seams and `commit-records` pass.

`PushAnchored` gains the `lockWait` parameter, takes the same lock and pushes the records side through the retry helper; a rejection that stays after the retry still returns `gitrepo.ErrPushRejected` unwrapped.
Its doc comment's "taking no lock is deliberate" reasoning is replaced: the lock is the weft-side `fabric.push.lock`, not `PushCoalesced`'s repo-root lock, and contending with the detached push child is the point, since the lock-free status push racing that child is the likely cause of #418.
State the bound there: only a waiter is bounded, and a holder lasts as long as git does.

The three existing `PushAnchored` callers pass `fabricengine.StatusPushLockWait`: battencli's `battenCommitStatusDeps` Push and its `PushSeed` closure in `wire.go`, and loomcli's `loomCommitStatusDeps` Push, which card 4 then moves to `PushPairAnchored`.
Batten's seam keeps pushing the records side only.

`doc.go`'s anchored entry-point paragraph names `PushPairAnchored`, `IsPushRejected` and the lock, and its "treat `gitrepo.ErrPushRejected` as a human-decidable condition rather than retrying" sentence states the one bounded retry instead.
The same file's list of vocabulary-neutral `l`-in entry points names `PushAnchored(l, opts)`; it becomes `PushAnchored(l, opts, lockWait)` and gains `PushPairAnchored(l, opts, lockWait)`.

What this lets through, and its bound: the retry pushes a second time without a human, only when the fetched remote tip is already an ancestor of or equal to local HEAD, once per side per call.
The bounded lock wait lets a status push give up without pushing; the run continues and the next transition retries.

Tests, extending `pushanchored_integration_test.go` on hubforge pairs (`hubforge.AddPair`, location from `lyxcwd.ResolveWorktree`), with a lock-path accessor added to `export_integration_test.go`:

- One scenario test for `PushPairAnchored`, steps in order on one pair:
  a raw `git commit` on each side is pushed, both remote branches sit at local HEAD, and the record holds one `KindBranchPushed` entry per side;
  a remote that rejects once (a `pre-receive` hook in the code side's bare that declines the first push it sees and accepts after) is retried and succeeds;
  a code side diverged on its remote (a second clone pushed a commit) returns an error naming the warp side and its worktree path that satisfies `errors.Is(err, gitrepo.ErrPushRejected)`, while the records side's new commit is still pushed and both local HEADs are unchanged;
  with the code side still diverged and the records side's origin pointing at a missing path, the error names both sides and still satisfies `errors.Is(err, gitrepo.ErrPushRejected)`.
- A table-driven lock test over both entries: with the push lock held by the test, a bounded wait returns an error satisfying `errors.Is(err, fabricengine.ErrPushLockBusy)` that names neither side, and no remote moves, for `PushPairAnchored` and for `PushAnchored`;
  `SkipPush` and `SkipGit` each return nil at once with the lock still held;
  `LockWaitUnbounded` returns only after the test releases the lock, then pushes both sides.
- `PushAnchored` gains its parameter in the existing tests, and `TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejectedUnwrapped` keeps pinning the bare sentinel after the retry.
  The scenario test's retry step also runs `PushAnchored` against a records remote that declines once, and the push succeeds on the second attempt; this is a row of that step, not a new test function.
- `TestPushAnchored_PushesAndRecordsBranchPush` also makes a raw commit in the pair's warp worktree and asserts, after `PushAnchored`, that the warp bare remote has not moved: this pins that batten's seam still pushes the records side only, as an assertion of the existing test.

**ImpactSummary:** `PushAnchored` gains a parameter and its three callers (two in `battencli`, one in `loomcli`) are all updated in this card; `PushPairAnchored` has no caller until cards 4 and 5; no other package's signature changes.

**Commit:** 3: fabricengine: both-sides anchored push under the absorbing lock, retrying once
