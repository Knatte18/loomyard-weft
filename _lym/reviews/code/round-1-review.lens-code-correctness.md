```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each added or changed branch on a concrete case.

**Per function and branch: case run, verdict**

- `runRecordsTracked` (`cleanup.go:260`):
  - Neither status nor seed tracked returns false. Correct.
  - Seed tracked, status not, returns true. Correct.
  - Error reading the status returns `(false, err)`. Correct, and the seed is not read.
  - Same result as the loop it replaced in `RecordsPresent` (`cancelcmd.go:241`).
- `removeRunRecords` (`cleanup.go:288`):
  - Run directory on disk and tracked: removed, committed. Correct.
  - Run directory gone and status tracked: commit runs, the retry path is kept. Correct.
  - Run directory gone and untracked, including the empty directory that `os.MkdirAll` in `RemoveRecords` recreates: `os.RemoveAll` deletes the empty directory, no `git add` runs, scratch is emptied, and the call returns `(false, nil)`. This fixes the reported failure. Correct.
  - Run directory on disk but untracked: removed with no commit. Correct. It used to fail on `git add`.
  - `deps.Committed` error returns a plain wrapped error, not a `removalCommitError`. The run directory is already removed and scratch is not emptied. `cancel` reports it with a re-run way forward. `cleanupSlug` skips the slug (`case err != nil`, `cleanup.go:227`). Correct, and the same class as the existing `RemoveAll` error.
  - `deps.Commit` error returns a `removalCommitError`, as before. Correct.
- `cleanupDeps.Committed`: `fabricCleanupDeps` sets it (`cleanup.go:76`), and the only other construction is `cleanupStubs.deps()` (`cleanup_test.go:55`). Every construction sets it, so there is no nil call. Correct.
- `HoldRemoteRefs` (`remoterefslock.go:25`):
  - `CommonDir` error returns before `Admit`, so nothing leaks. Correct.
  - `AcquireWriteLock` error calls `leaveGate()` and returns. Correct.
  - Success returns a release that runs `held.Release()` and then `leaveGate()` whatever `Release` returns. Correct.
  - `judgeAgainstOrigin` takes two holds. The second `Admit` passes at once because `admissions` is already above 0, and the deferred releases unwind in LIFO order with the counter back to 0. Correct.
- `Admit` (`hold.go:424`):
  - With no gate, `Network` returns a no-op, and `admissions` is still incremented and then decremented. Balanced. Correct.
  - Called while a pause covers the process with no admission standing: waits at the gate as `Network` does, the intended behavior.
  - Called while an admission already stands: passes at once and writes a marker, so it cannot wait forever.
  - The marker is removed by `done()` in the release, so no stale in-flight marker remains. Correct.
- `Network` (`hold.go:382`) with an admission standing: the early return skips the `Defer` handoff and the waiting marker. Correct, because the first `Admit` already did the handoff check at entry. The `held` it returns is 0. The `gitexec` call site ignores `held`; the `githubclient` call site (`transport.go:47`) uses it and was not read.
- `markInFlight` (`hold.go:434`): same marker fields as before, including `UnderParentLock`. A marker write failure returns a no-op. Correct.

**Findings**

1. LOW, `design`, `internal/pauserecord/hold.go:387`.
   - `admissions` is process-wide, so while one goroutine holds the remote-refs lock, every `Network` call of the process passes the gate. That includes calls unrelated to the lock: a GitHub call (`githubclient/transport.go:47`) or a fetch in another repository.
   - The window is one git call for `withRemoteRefs`. For `judgeAgainstOrigin` (`originreconcile.go:761`) it spans the `ls-remote` retries and sleeps.
   - A pause that starts in that window does not hold the process's other calls. Only the process's own in-flight markers show them.
   - `Admit`'s doc says "every network call of the process", so it is intended, but it widens a pause's reach beyond the lock holder.
2. LOW, `design`, `internal/battencli/cleanup.go:261`.
   - Tracking is judged from the status and the seed only, but the commit covers the whole `RunDirRel`.
   - If the records HEAD tracks some other file under the run directory and neither status nor seed, the removal commits nothing and the file stays tracked.
   - `RecordsPresent` and `judgeRunDirectory` read the same files, so the state is consistent with them. I could not confirm whether a run directory can reach it.

No wrong result or swallowed error found in the added or changed branches, and no inconsistent state on the failure paths I ran.

Tool calls: none; everything needed was in the inherited context.
