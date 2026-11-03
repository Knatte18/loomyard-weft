```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1–F5 change set (commit `fa3ac4cd`) for code correctness.

I found one LOW finding, and no BLOCKING or MEDIUM findings. I traced these cases by hand:
- **F1:** `RepoCode` and `RepoRecords` are untyped constants equal to the in-package values, so `parentOutcome` and `stoppedBranches` compare the same strings the reconcile sets. The grep found no leftover loomcli `codeRepo` or `parentNoticeFile`. Its remaining hits belong to `reportcli` and `tools/tokencount`, which are unrelated.
- **F2:** `ParentNoticeFile` returns the same path the old inline join produced. `filepath` is still used by `announcedBefore`, and the gate compiled.
- **F4:** The own busy records branch has `Repo == RepoRecords` and `State == BranchBusy`, so it is skipped and the stop reads "a branch every run shares". With the skip removed, `BranchBusy.Stops()` is true, the notice would name `task-records (busy)`, and the `StoppedSharedBranchBesideOwnBusyRecords` row would fail.
- **F3:**
  - With the `guarded*` wrapper dropped from `Commit` or `Push` in `commitStatusDepsGuardedBy`, or from `PushTask` in `pullRequestDepsGuardedBy`, the real work would run against the synthetic location. The error would not wrap `errPushStopped`, so `TestPrePushCheckWiredIntoDeps` would fail.
  - `TestNewPrePushCheckSkip` fails if `skip` stops reading `EnvSyncOptions`, or if the prime read is dropped.
  - The subtests do not call `t.Parallel()`, matching the `t.Setenv` rule.

**LOW — `pattern/PATTERN-owner-machine.md:6`, the rewritten bullet.**
- **What is wrong:** It lists "pushes off" as a skip, but `skipsCheck` also skips when `opts.SkipGit` is set (`prepush.go`, `skipsCheck`), and the `TestSkipsCheck` row `GitOff` pins that. The bullet is meant to match `prepush.go`'s skips, and it still under-states them by one.
- **Fix:** Write "with git or pushes off".

**LOW — `TestNewPrePushCheckSkip` in `prepush_test.go`, the `skip` closure's last two reads.**
- **What is wrong:** The test has no case for the pause-coverage read or the producer read. Its pair case expects false. A closure that always returns an empty producer, or ignores `pauseCovered`, still passes.
- **Evidence:** The closure passes `shedrun.StatusFile` and `shedrun.StatusLock` with `runID`, and no row writes a status file at Finalize or a pause record. The review's F3 asked only for the pushes-off and prime cases, so the stated premise is met.
- **Fix:** Optionally add a row with a Finalize status file.

Further tool calls:
1. `grep -rn "codeRepo\|parentNoticeFile"` over `.go` and `.md` files, to confirm no leftover reference to the deleted loomcli constants.
