---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: internal/gitrepo/remoterefslock.go:11
    summary: The diff adds a new repository-wide lock file instead of taking the push lock the amended entry names, and no parent decision records the departure
    fix: Leave the code in place and defer this finding to the run's parent. The parent records the new lock as the settled way with `lym loom decision add`, or refuses it. That call lies outside the fixer's write surface, so the fixer changes nothing.
  - id: F2
    severity: LOW
    class: scope
    location: internal/fabricengine/originreconcile_integration_test.go:578
    summary: No test asserts that the remote-refs locks are free after the reconcile returns an error from a failed fetch or from a failed branch judgment
    fix: In the existing subtest "A fetch that fails after both ls-remote calls answered is an error and moves nothing", take `lockPaths := remoteRefsLockPaths(t, hub)` before the call and, after the error assertions, fail when `len(freeRemoteRefsLocks(t, lockPaths)) != len(lockPaths)`. Add this as an assertion in that subtest, not a new test, so the fetch-error return is pinned beside the three returns the "frees both remote-refs locks on every return" subtest already pins.
---

### [MEDIUM] Departure from the push lock the amended entry names is unrecorded

**Location:** internal/gitrepo/remoterefslock.go:11, internal/fabricengine/originreconcile.go:759
**Issue:** The entry's `Idea` says every lym fetch "takes the lock that already orders that repository's pushes (the push lock)".
The diff instead adds `lym-remote-refs.lock` in the common git dir (`RemoteRefsLockFileName`) and leaves `.gitrepo-push.lock` and the fabric push locks untouched.
`_lym/landing/summary.md` argues the departure under "Design decisions": the existing push locks are per checkout, so they do not order two worktrees of one repository, and `PushCoalesced` holds its lock across a loop that fetches, so reusing it would deadlock in-process.
I checked that against the code. `PushCoalesced` takes the per-checkout `.gitrepo-push.lock` and then calls `pushWithRebaseRetry`, so the argument holds and the new lock is sound.
The decision record carries no `Added after Discussion` entry that settles the choice, so it is an unrecorded deviation from the amended entry.
Only the parent can record or refuse it, so the fixer leaves the code alone and defers.

### [LOW] Lock release is not pinned on the fetch-error and judge-error returns

**Location:** internal/fabricengine/originreconcile_integration_test.go:578
**Issue:** `judgeAgainstOrigin` frees both locks through one `defer` per lock, so every return frees them today.
The subtest "A reconcile frees both remote-refs locks on every return" pins the judge-only, unreachable-origin and ls-remote-refusal returns.
The returns after a failed `fetchMovedHeads` and after a failed `judgeBranch` have no lock assertion.
The existing fetch-failure subtest already drives the first of them and only skips the lock check.
A later edit that swaps the `defer` for explicit releases on the success path would leak both locks on those returns and pass every test.
The `judgeBranch` error return stays unpinned; it needs a forced git failure, and the shared `defer` makes it the same path, so the fetch-error assertion is the cheap pin.

## Focus answers

- **Fetch, pull and push waiting on the held lock.**
  `TestRemoteRefsLock_EveryFetchPullAndPushMethodWaitsWhileTheLockIsHeld` holds the lock, starts each of `Fetch`, `FetchWithArgs`, `Pull`, `Push`, `PushRebaseFree`, `DeleteRemoteBranch`, `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased`, and fails if the call returns within the window.
  Each row's arguments pass the method's early input checks, so a method without the lock returns at once and the row fails.
  A method that dropped the lock fails the test unless its git call outlasts the 500 ms window, so a slow machine can miss a regression but never fail a correct method. I accepted that as the cost of proving a wait.
- **Locks held from the origin read through every verdict.**
  Both holds sit in `judgeAgainstOrigin` and release only when it returns, after the last `judgeBranch`.
  `originVerdictHook` runs before each verdict, and the subtest "holds both repositories' remote-refs locks across its ls-remote calls, fetches and branch verdicts" fails if either lock is free there, so releasing after the fetch fails the test.
  Neither `fetchMovedHeads` nor `judgeBranch` calls a locking `gitrepo` method, so the non-reentrant lock cannot deadlock against itself.
- **Every return of `ReconcileOrigin` frees the locks.**
  The locks live only in `judgeAgainstOrigin`; `ReconcileOrigin` runs its teardowns and moves after it returns, so no later return holds them.
  Tests cover the judge-only, unreachable-origin, ls-remote-refusal and success returns; F2 covers the rest.
- **`PATTERN.md` against `archive.go`.**
  The entry's "a tag push ... write none" matches `tagAndPushArchiveTip`, which pushes `refs/tags/...` and writes no tracking ref.
  `resolveArchiveTip` no longer reads `FETCH_HEAD`: it takes the tip from `ls-remote`, fetches that SHA through `FetchWithArgs`, and returns the SHA, so no read happens after the lock is released.
  The subtest "a branch only on origin is archived from the fetched tip" drives that path.

## Rejected

- The ls-remote retry sleeps run while the locks are held, which delays other fetches and pushes while origin is unreachable. The record asks for the read and the verdict under one lock, `judgeAgainstOrigin` documents the bound, and a writer that waited would face the same unreachable origin.
- The fetch-by-SHA in `resolveArchiveTip` is a behavior change from fetching the ref. It fixes the `FETCH_HEAD` read, the SHA comes from an `ls-remote` seconds earlier, and the origin-only archive subtest passes it.
- The by-hand method list in the gitrepo waiting test, and the guard that scans only `internal/fabricengine`, do not catch a future `gitrepo` method that forgets the lock. The record asks for the existing fetches and pushes to be locked, and all are.
- The summary's claims match the diff, including the list of fabricengine fetches moved to `FetchWithArgs` and the tests it names.
