---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/loomcli/smoke_test.go:463
    summary: killDriversAndClearLog returns once the locks are free, but a git child the killed driver was running can still hold the records index lock
    fix: In killDriversAndClearLog, after waitRunLockFree, also wait until the records worktree's index.lock is gone, by polling for the file at the path git rev-parse --git-path index.lock names in fabricengine.RecordsWorktree(loc) with a short deadline, so the git rm in the self-heal subtest cannot race a git commit the driver started before it was killed.
---

### [LOW] killDriversAndClearLog does not wait for the driver's in-flight git child

**Location:** `internal/loomcli/smoke_test.go:463`

**Issue:** The publish failure is a live driver committing into the records worktree while the self-heal subtest runs `git rm`; the log shows `index.lock: File exists` at 0.00s into the subtest.
HEAD already adds `killDriversAndClearLog(t)` at the top of that subtest (f64b4a74d), and it settles the focus question.
The subtests before it ("concurrent first bootstraps", "bootstrap leaves one status strand", "second invocation") leave a first-bootstrap driver alive, and that driver commits status into the same records worktree through `newCommitStatusSeam`.
The subtest now stops every such driver, as the "died driver proceeds" subtest does.
That is the same helper, so the two subtests are consistent, and the kill covers the driver and the `loom step` processes it spawned.

One narrow window stays open.
`killGoDriver` sends SIGKILL through `proc.KillPID`, which does not wait for exit, and the git process a driver runs for `Commit` is neither a `loom drive` nor a `loom step` process, so it is not killed.
`waitRunLockFree` only observes the run and driver locks, which the kernel frees the moment the driver dies, not the git child.
A driver killed mid-commit can therefore leave a git child holding `index.lock` for a few milliseconds after the helper returns, and the next `git rm` can still hit it.
The window is small and the helper change closes the failure the verify run recorded, so this is LOW.
I could not run the tmux-tier test here, so the finding rests on reading the code and the verify log, not on a reproduction.

## Rejected

- Comment convention on the added comment in the subtest: it is a why-comment, one sentence per line, and complies.
- Test economy: the change adds no test, only a helper call in an existing subtest.
