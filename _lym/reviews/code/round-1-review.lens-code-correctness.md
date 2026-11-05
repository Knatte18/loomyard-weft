```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the lock-move diff, running each changed function against concrete inputs.

## Function-by-function list (case run → result)

- `lock.TrackedLockPath(root, name)`: ran a root with a symlinked ancestor, and a root that does not exist. Correct. The symlinks resolve, a missing root is used as given, and the result is `<root>/.lym/locks/<name>`.
- `lock.TrackedFileLockPath(path, suffix)`:
  - `/a/_lym/reviews/code/x.json` → mirrored under `/a/.lym/locks/_lym/reviews/code/x.json.lock`. Correct.
  - Relative `_lym/reviews/x.json` → `.lym/locks/_lym/reviews/x.json.lock`, relative to cwd. Correct.
  - Bare `x.json` or relative `reviews/x.json` with no `_lym` → terminates at `.` and returns the file with the suffix beside it. Correct, no infinite loop.
  - `_lym` as a junction → the lock lands in the holding checkout. Correct.
  - Absent `_lym` directory → falls back to the resolved parent. Correct. The lock path changes once the junction appears, which is only a transient mismatch.
  - Nested `_lym` → the nearest one wins. Correct.
- `lock.checkoutHolding` and `resolve`: an `EvalSymlinks` failure falls back to the given path. Correct.
- `lock.refuseTrackedTree`: ran `/r/_lym/reviews/x.lock`, `/r/.lym/locks/_lym/x.lock`, and a path with no lym segment.
  - The first is refused. The second and third are accepted. Correct.
  - Finding 1 below covers the first-`.lym`-wins bypass.
- Five `Acquire*`/`TryAcquire*` wrappers: the refusal runs before any lock file is created, so the refused case leaves no file. Correct. `TestAcquire_LockInTrackedLymDir` pins this.
- `state.mkdirParents` and the `WriteJSON`/`WriteFile`/`UpdateJSON` callers: a lock directory absent on first write is created before the lock is taken. Correct. A failed mkdir returns before any lock is held.
- `state.ReadJSON` with the lock directory absent: `AcquireReadLock` fails with `ErrNotExist`, and `ReadJSON` reads unlocked. Correct. `state.go:105-115` was already unchanged by the diff. This answers the question of a read on a fresh checkout whose `.lym/locks` does not exist yet.
- `boardengine.swapLockPath` (`store.go:30`): `board.json` in `/b` → `/b/.lym/locks/board.json.swaplock`. Correct. `Load`, `readLegacyFile`, `Save` and `ReadOwners` all use it.
- `fabricengine.BoltLockPath` and `BoardWriteLockPath`: the directory is created, and an mkdir failure returns an error naming the Bolt directory. Correct.
  - `tryRecordsLocks` with a non-existent `worktree` now creates that directory tree on the Bolt branch. This is an edge case, not a finding.
- `Bolt.acquireWriteLock` and its three callers: the error is returned once, with the wrap added only in the lock-acquire branch. Correct, with no double wrapping.
- `Bolt.Sync`, `pullFirst`, `retryBoltPush`, `tryRecordsLocks` (Bolt branch):
  - `Repo.PushLockPath` returns an error when the checkout is not a repository, and every caller propagates it. Correct.
  - The lock order, write lock then push lock, is unchanged.
  - `repo` is now created before the push lock, with no behavior change.
- `Repo.PushLockPath` and `PushCoalesced`: a linked worktree locks in its own `.git/worktrees/<n>` directory, which matches the old per-worktree-root scope. Correct.
- `boardengine.ensureBoardIgnores`:
  - `""` → `".board-rendered.json\n"`. Correct.
  - `"*.lock\n*.swaplock\nM\n"` → `"M\n"`. Correct.
  - `"a"` with no trailing newline and no manifest line → `"a\n"` plus the manifest line. Correct.
  - CRLF input is trimmed for matching. Correct.
  - Unchanged content → no write. Correct.
- `fabricengine.seedWeftArtifactExcludes`: removes the retired wildcard lines, then appends the missing entries (`.weft/`, `.lym/`, the legacy names). A removal-only change is still returned as new content. Correct. It is idempotent, and `TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent` pins that.
- `shedadapters` `readRecord`, `writeRecord`, `writeSettleRecord`, `CarryRecords`: the lock sits in the mirrored lock directory, which `WriteFile` and `WriteJSON` create. The read side covers an absent lock directory through `ReadJSON`. Correct.
- `heddlecli.saveLocked`, `heddleengine.read`: the `BoltLockPath` error is returned and wrapped. Correct.

## Findings

1. **LOW / design**, `internal/lock/tracked.go:70-79`, `refuseTrackedTree`.
   - The loop returns nil at the first `.lym` segment of the whole path, even when a tracked `_lym` segment follows it.
   - Input: a checkout under a directory named `.lym` (such as `~/.lym/x`) with a lock at `<checkout>/_lym/reviews/x.lock` → accepted. The guard only holds when no earlier ancestor is named `.lym`.
   - The guard also misses a `_lym`-segment lock path that has a `.lym` segment elsewhere before it.
   - Evidence: `tracked.go:70-79`.

2. **LOW / design**, `internal/lock/lock.go:32-35` and `internal/fabricengine/junctionnames.go:100-107`, the upgrade overlap.
   - The Bolt write, push and swap locks, and the review-record locks, moved to new paths.
   - A binary from before the commit and one after it hold different lock files for the same board during an upgrade, so they do not exclude each other.
   - Neither the diff nor the commit message names a reconcile or a drain step for this.
   - This is consistent with how the old locks stay excluded by name, but the overlap window is real.

I found no other defect in this lens.

## Tool calls

1. Bash: read `state.go` lines 68-135 (`ReadJSON` and `readJSONUnlocked`) and `recordsgit.go` lines 96-135. The diff did not show `ReadJSON`'s missing-lock-directory handling or the end of `seedWeftArtifactExcludes`.
2. Bash: grepped `emptyScratch` callers, read `failedpush.go` lines 85-106 and `ownerset.go` `ReadOwners`, and grepped read-lock use. This checked whether a scratch cleanup could reach the Bolt lock directory, the failed-push lock location, and the `ReadOwners` read-lock call.
