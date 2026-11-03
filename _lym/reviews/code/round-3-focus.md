---
round: 3
focus:
  - "Detached handover coverage: `spawnTeardownHold` in internal/fabricengine/remoteteardown.go starts the detached teardown child, and `holdTeardown` takes the in-process branch whenever `testing.Testing()` is true. Settle whether any test at any tier exercises the spawn path's answer protocol (held, busy with and without try, error or closed stdout), the lock file recorded in `rec`, and the registration under `runproc.RoleTeardown`, and whether `teardownChildCmd`'s `RunE` in internal/fabriccli/teardownchild.go is run by any test beyond its arity and role-guard tables."
  - "Notice coverage of the two paths that end a remote teardown job early: `warnWhenEnded` in internal/fabriccli/teardownchild.go (signal mid-job) and `childHold.finish` in internal/fabricengine/remoteteardown.go (job write fails). Settle whether each reaches the creel notice as well as the fabric log, and whether `postRemoteTeardownNotice` has a row for each."
  - "Way-forward text in internal/fabricengine/remoteteardown.go: `noticeUnreadRemoteTeardown` for a job whose kind is unknown, and `failedRemoteSteps` and `runRemoteTeardown` when the weft repo root or `job.Worktree` cannot be resolved after the archive succeeded. Settle whether each way forward names a command that retries the step that actually failed, for every job kind (removal, cleanup, task-branch discard)."
---

The focus targets the open detached-handover key and the new notice and way-forward text introduced by the round-2 fixes.
