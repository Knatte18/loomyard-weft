---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/lyxtemp/sweep.go:68
    summary: removeRunDirectory retries back to back with no pause, so the retry can land inside the peer probe's hold of the recreated lock file on Windows
    fix: In removeRunDirectory, sleep a short fixed interval (a package constant such as removeRetryDelay, a few milliseconds) between failed attempts, but not after the last one; keep the not-exist-is-success check and the attempt cap.
---

The Publish failure is `TestSweep/two_concurrent_sweeps_both_succeed`, which failed with `directory not empty` on `rdead00`.
The cause is `ownerDead`: its `lock.TryAcquireWriteLock` opens the lock file with create semantics, so a probe that loses the race to the peer's `os.RemoveAll` recreates the lock file and the peer's final rmdir fails.
`removeRunDirectory` (commit 7cfcaa99c) answers this with a bounded retry that counts a path already gone as removed.
I judge it correct on the evidence below.

- Each sweeper probes a given directory once, so with two sweepers the lock file is recreated at most once per directory.
  A retry after that recreation finds the directory with nothing recreating the file again, so the removal converges within the cap.
- A path the peer removed first makes `os.RemoveAll` return nil, and a failed attempt followed by `Lstat` reporting not-exist returns nil; neither lands in `Failures`.
- A probe that opens the lock file after the directory is gone gets `ErrNotExist`, which `ignoreNotExist` turns into "not dead, no error".
- `TestSweep/two_concurrent_sweeps_both_succeed` is unchanged: it still asserts no failures from either sweep and an empty root.
  `lyx gate test ./internal/lyxtemp` passes.
- The `ownerDead` and `Sweep` doc comments match the behavior: the probe may recreate a removed lock file and releases its own hold before returning, and a vanished entry is no failure.

### [LOW] Back-to-back removal retries

**Location:** internal/lyxtemp/sweep.go:68
**Issue:** The retry loop re-runs `os.RemoveAll` immediately.
On Linux the recreated lock file can be unlinked while the peer holds it, so the immediate retry converges.
On Windows, which the repository targets, a file the peer's probe holds open generally cannot be deleted, so all attempts can run inside the probe's brief hold and report a failure that a short pause would avoid.
The race is narrow and the cap bounds the damage, hence LOW.

## Scope note

The range is the whole branch over `main`, 171 changed files.
I judged the focus area in depth and did not re-derive the remaining cards' mechanical checks line by line, so this APPROVED rests on the focus questions settling cleanly and on no other defect surfacing in the files I read.
