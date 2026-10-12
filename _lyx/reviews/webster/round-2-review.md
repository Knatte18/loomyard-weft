---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/lyxtemp/sweep.go:50
    summary: TestSweep/two_concurrent_sweeps_both_succeed fails because a sweeper's lock probe recreates the lock file inside a directory the other sweeper is removing
    fix: In `Sweep`, when `os.RemoveAll(path)` returns an error, retry `os.RemoveAll(path)` a small fixed number of times (three attempts in all) before appending the `SweepFailure`, and treat a path that no longer exists (`os.Lstat` reports `fs.ErrNotExist`) as success rather than a failure. Update the `ownerDead` doc comment, which says the probe never creates the lock, to say the probe may briefly recreate a lock file a concurrent sweeper has just removed. Keep `TestSweep/two_concurrent_sweeps_both_succeed` as the regression test; do not weaken its assertions.
---

### [BLOCKING] Concurrent sweeps race on the lock probe and fail with "directory not empty"

**Location:** internal/lyxtemp/sweep.go:50 (`Sweep` removal) and internal/lyxtemp/sweep.go:80 (`ownerDead` probe)

**Issue:** Publish's `publish_verify` failed on `TestSweep/two_concurrent_sweeps_both_succeed` (log `.lyx/verify/verify-4.log`): sweep 1 reported `unlinkat .../rdead00: directory not empty`.
The package arrived through the merge-in commit `0e95f8d72`; the merge's diff adds `internal/lyxtemp` whole, and the failing code is in that merge, not in the branch's own commits.
The cause is a check-then-act race between the two sweepers:

1. Sweeper B `Lstat`s `rdead00/<lock>` and finds it.
2. Sweeper A's `os.RemoveAll` deletes the lock file.
3. B calls `lock.TryAcquireWriteLock`, which wraps `flock.TryLock`; flock opens with `os.O_CREATE|os.O_RDWR` (`flock_unix.go:187` in `gofrs/flock@v0.8.1`), so B recreates the lock file in the directory A is still removing.
4. A's final `rmdir` sees the recreated file and fails with `ENOTEMPTY`.

`Sweep`'s doc says an entry vanishing under a concurrent sweeper is no failure, and `ownerDead` says the probe never creates the lock; neither holds, because the not-exist handling covers only `ENOENT` and the probe can create.
B's probe then releases and B's own `RemoveAll` would remove the leftover, so a retry converges.
The failure is timing-dependent, but the package's own test pins this behavior and it failed in the gate, so it blocks the publish.

I read the branch's own range (`git diff e6e0baf..HEAD`, no live rework round exists) only as far as the stat and the commit list; the failing test is the finding this round settles.
