```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Each function or branch below is listed with the case I ran.**

- `Remove` handover (`remove.go:210-222`, `:365-399`): a failure after the lock is correct. `finish(nil)` closes the child's stdin and waits for it to release the lock, and only then does `respawnPairPush` run.
- `Remove` handover (same lines): a success is correct. The parent holds no lock, so there is no gap, because the child answers `held` before the first mutation and keeps the lock through the last step.
- `spawnTeardownHold` (`remoteteardown.go:240-286`): correct for `held`, `busy`, an `error ...` line and EOF. The lock file is recorded in `rec` in each case.
- `RunTeardownChild` (`remoteteardown.go:293-329`): correct for an empty job, which releases the lock at once, and for a parent that crashes, which gives EOF and releases it. A job that fails to unmarshal is not correct; see finding 3.
- `runRemoteTeardown` step order (`remoteteardown.go:99-126`): weft-tip push, then tag push, then leased weft delete, then task-branch delete. This is correct, and an archive error skips the later steps and is returned.
- `runRemoteTeardown`, `DeleteWeftOnOrigin` (`remove.go:374`, `remoteteardown.go:110-121`): correct for a weft branch that is local-only, origin-only, kept locally, or refused on push. A refused push leaves `leaseTip` empty, so no delete happens.
- `archiveWeftTip` (`archive.go:133-148`): correct. `PushCommit` pushes by SHA after the local branch is gone, and the objects survive under the local tag.
- `noticeRemoteTeardown` (`remoteteardown.go:130-165`): the archive failure and the weft-delete failure are correct. Origin task-branch failures are not reported; see finding 1.
- `DiscardTaskBranch` and `discardTaskBranchOnOrigin` (`taskbranchdiscard.go`, `remoteteardown.go:344-373`): correct for no local branch, no origin branch, local contained in origin, and origin contained in local. The `job == nil` defer releases the lock on every early return.
- `cleanupOrphan` (`cleanup.go:272-329`): correct for lock busy, lock error, a worktree now live, archive skipped (no origin), a failed local delete, and in-process and handed-off outcomes.
- `add.go` parallel probes (`add.go` diff, `:343-380`): correct. The warp `ls-remote` goroutine runs beside the weft probe, `Wait` runs even when the weft probe fails, and the judge runs after the weft answer. The two probes use different repositories.
- `statusPushRequest` (`spawn.go`): correct. `WeftPath` is never empty, so spawning unconditionally is right now that the local skip is gone.
- `pushPlainWithRetry` and `PushCommit` (`push.go`): correct. `source` is ignored when `branch` is empty, and an empty `source` falls back to `refs/heads/<branch>`.
- Push notifier in the child: correct. `cmd/lym/main.go:156` installs it for every command, and the child starts in the Bolt worktree, which `InstallPushNotifier` resolves.

**Findings**

1. **MEDIUM, `design`, `remoteteardown.go:150-155` with `remove.go:593-609`.**
   - A failed origin task-branch delete is never mailed or warned about.
   - `deleteTaskBranchOnOrigin` returns `ls-remote`, fetch and delete failures as `keptReason`, so they land in `out.taskKept`.
   - `noticeRemoteTeardown` has no case for `taskKept`, and the default branch only logs `Info "remote teardown done"`.
   - The operator decision says every async remote failure goes to the fabric log and a creel notice.
   - The `switch` also reports only the first failure. A task-branch failure that follows a weft-delete `RemoteError` is dropped.

2. **LOW, `design`, `remoteteardown.go:131-149`.**
   - The default way forward is ``run `lym fabric remove <slug> --remote` again``. It is wrong for `cleanup` jobs, where the command that failed was `lym fabric cleanup --apply --remote`.
   - It is also wrong for a `remove` whose origin branches were never pushed. A re-run then returns `ErrPairNotFound` ("nothing remains"), because `refuseWhenNothingRemains` finds no local remnant and no origin tip.

3. **LOW, `design`, `remoteteardown.go:317-320`.**
   - A job that fails to unmarshal drops every remote step, and the failure is neither mailed nor named with a way forward.
   - `RunTeardownChild` returns the error, and the child verb only logs a `Warn`.
   - The same gap exists when `io.ReadAll` fails.

4. **LOW, `consistency`, `remove.go:46-51` and `remove.go:261-263`.**
   - `RemoveStepArchive` is documented as "the archive tag pushed to origin", but the step is now appended when the tag is only created locally.
   - With a hand-off the push may later fail, so the step list overstates what happened. The doc comment still claims "appended only after it changed state".

**Tool calls**
1. grep for `installedPushNotifier` and the install functions: my context lacked where the notifier is set.
2. grep for `orchcli.InstallPushNotifier` callers and a read of `pushnotifier.go`: my context lacked whether the detached child process gets a notifier.
