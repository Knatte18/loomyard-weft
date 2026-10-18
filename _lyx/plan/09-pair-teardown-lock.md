# Card 9 — pair-teardown-lock

**Create:**
- `plan:internal/lock#TryAcquireReadLock` -> `func TryAcquireReadLock(lockPath string) (*FileLock, bool, error)`
- `plan:internal/lock_test#TestTryAcquireReadLock` -> `func TestTryAcquireReadLock(t *testing.T)`
- `plan:internal/fabricengine#TeardownLockPath` -> `func TeardownLockPath(hubPath, slug string) string`
- `plan:internal/fabricengine#DetachedPush` -> `func DetachedPush(hubPath string, req PushSpawn) (PushResult, error)`
- `plan:internal/fabricengine#ErrPairPushRunning` -> `var ErrPairPushRunning error`
- `plan:internal/fabricengine#SetTeardownWaitForTest` -> `func SetTeardownWaitForTest(t *Topology, wait time.Duration)`
- `plan:internal/fabricengine#detachedPushHeldHook` -> `var detachedPushHeldHook func()`
- `plan:internal/fabricengine#removeAfterTeardownHook` -> `var removeAfterTeardownHook func()`
- `plan:internal/fabricengine#SetTeardownHooksForTest` -> `func SetTeardownHooksForTest(t *testing.T, heldByPush, afterRemoveTeardown func())`
- `plan:internal/fabricengine_test#TestRemove_TeardownLockAgainstThePairsPush` -> `func TestRemove_TeardownLockAgainstThePairsPush(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology`
- `internal/fabricengine#NewTopology`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#StatusPushLockWait`
- `internal/fabriccli#addWeftVerbs`
- `internal/battencli#teardownRefusal`
- `internal/battencli#TestTeardownRefusal_RecordsRemedyNamesCommitRecordsNeverForce`
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#PushSpawn`
- `plan:internal/fabricengine#CoalescePushRequest`
- `plan:internal/fabricengine#SpawnStatusPush`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#SwapPushRecorderForTest`
- `internal/lock#AcquireWriteLockWithin`
- `internal/lock#AcquireReadLock`

**Intent:** Exclude `Remove` against the pair's own detached pushes through a per-slug teardown lock outside both of the pair's worktrees (D1's teardown paragraphs).

`TryAcquireReadLock` is `AcquireReadLock`'s non-blocking form: a shared lock, false with a nil error when an exclusive holder has it.
`TeardownLockPath(hubPath, slug)` is `<HubScratchDir(hubPath)>/teardown/<slug>.lock`, outside both worktrees, so neither the lock file nor any child's working directory or log file sits in a tree `Remove` deletes.

`DetachedPush(hubPath, req)` is the detached child's body, which `fabriccli`'s push bypass now calls with the hub its `_board` working directory resolves to:
- with `req.TeardownSlug` set, it creates the lock's directory and takes the teardown lock shared through `TryAcquireReadLock` for its whole run, and exits with a Warn ("the pair is being removed; its push is not run") and a nil error when an exclusive holder has it;
- every child, teardown slug or none, then checks that each worktree path it was given is an existing directory before it creates the push lock's directory or touches anything, and exits with a Warn and nil when one is gone; a child with a teardown slug checks while holding the lock, so it leaves nothing at a removed path;
- it then runs `CoalescePushRequest(req, SyncOptions{})`.

`Topology` gains `teardownWait`, which `NewTopology` sets to `StatusPushLockWait` and `SetTeardownWaitForTest` shortens.
`StatusPushLockWait`'s comment, and `doc.go`'s sentence on it, now say it is `commit-records`' push-lock wait and the default of `Remove`'s teardown-lock wait, the per-transition status pushes no longer passing it.
Two hooks, nil in production and set only by a non-parallel test through `SetTeardownHooksForTest` (in an export test file): `detachedPushHeldHook` runs inside `DetachedPush` while it holds the teardown lock shared, and `removeAfterTeardownHook` runs inside `Remove` right after the worktree teardown, before it releases the lock.
`Remove` takes the teardown lock exclusive through `AcquireWriteLockWithin(TeardownLockPath(l.HubPath, slug), t.teardownWait)` after `RemoveRefusal` and the pair inspection, and before its first mutation, local path included; it releases it after its last step, past the worktree teardown and the store housekeeping.
When the wait runs out it returns, before any mutation, an error wrapping `ErrPairPushRunning`: "a push of the pair %q is still running or waiting under a pause; re-run `lyx fabric remove %s` once the push finishes or the pause is resumed, or end the stuck push child, whose pid its spawn logged".
The input reaching it: a child of the pair holding the shared lock past the wait; `TestRemove_TeardownLockAgainstThePairsPush` reaches it.
A `Remove` that fails after taking the lock and leaves the pair's task worktree in place spawns the pair's push through `SpawnStatusPush` after releasing the lock, so a child that exited on the held lock loses nothing.
A push of the prime's records side carries no teardown slug (card 7), so batten's status push spawned at the transition into `Worktree-Teardown` neither delays nor refuses that row's `Remove`.
`pairteardown` gets this refusal from `Remove` after its own `RemoveRefusal` probe passed, unchanged, and batten's `Worktree-Teardown` producer stands Stuck on any `Remove` error, as `battenshed`'s teardown producer already does.
`teardownRefusal` maps an error wrapping `ErrPairPushRunning` to "the pair for %q was left in place because a push of it is still running; once it finishes, resume this run with \"lyx batten run %s\"", wrapping the error.
What this lets through: nothing new; it adds a wait and a refusal to `Remove`; what bounds the wait: `teardownWait`, `StatusPushLockWait` in production.
A child run by hand with a wrong or empty teardown slug takes no teardown lock of the pair and can race a removal, the bound D1 states.

Tests:
- `TestTryAcquireReadLock`: a second shared holder acquires, an exclusive holder makes it return false with a nil error, and after the exclusive release it acquires.
- `TestRemove_TeardownLockAgainstThePairsPush`, serial (it sets the package hooks), over a hubforge hub with one ordinary pair per row, every wait shortened with `SetTeardownWaitForTest` and every ordering told by channels, never a sleep:
  - a shared teardown lock held by the test past the wait makes `Remove` return an error wrapping `ErrPairPushRunning` naming `lyx fabric remove`, with the task and records worktrees still present and no `archive/` tag;
  - a `DetachedPush` of the pair parked in `detachedPushHeldHook` makes a `Remove` with the shortened wait refuse with `ErrPairPushRunning`; once the hook is released and `DetachedPush` returns, `Remove` succeeds, so a child holds the lock shared for its whole run;
  - with `removeAfterTeardownHook` starting a `DetachedPush` of the pair's paths, that push returns nil having taken no lock and pushed nothing, so `Remove` still holds the lock past the worktree teardown;
  - a `DetachedPush` of the pair's paths after `Remove` with `remote` true returns nil, recreates nothing at either removed path, `ls-remote` on each origin lists neither of the pair's branches, and a following `Add` of the same slug succeeds;
  - with the weft `origin` URL pointing at a missing path, `Remove` fails at its archive after taking the lock, leaves the pair, and, with `SwapPushRecorderForTest`, records one push request for the pair with both slugs, made after the lock was released.
  The rows use no platform branch, so the same test pins the Windows half of the criterion where the suite runs on Windows.
- `TestTeardownRefusal_RecordsRemedyNamesCommitRecordsNeverForce` gains a row: an error wrapping `ErrPairPushRunning` maps to the text naming `lyx batten run <slug>` and still satisfies `errors.Is(err, fabricengine.ErrPairPushRunning)`.

`internal/fabricengine/doc.go` states the teardown lock, its place outside the pair, and the order of `Remove`'s steps around it.

**ImpactSummary:** Every `Remove` now takes one lock file under the hub scratch directory before mutating; with no child holding it the call costs one lock acquisition, and its callers (`pairteardown`, `lyx fabric remove`, batten's teardown) see one new refusal.

**Commit:** 9: fabricengine: a per-slug teardown lock between Remove and the pair's pushes
