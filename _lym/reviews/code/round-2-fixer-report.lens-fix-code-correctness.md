```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I ran each changed function and branch on a concrete input and found no wrong result, no broken caller and no vacuous test.

**Traced**
- **`spawnTeardownHold` and the stand-in child.**
  - The binary's args are `[exe, standin, mode, slug]`, which is four, so the `TestMain` guard matches.
  - The stand-in's cwd is `BoltDir(hub)`, so `filepath.Dir(cwd)` is the hub.
  - It writes the lock file before it prints `held`, so `existed` is false and `pathPresent` is true when the parent reads the answer. The recorded target equals `hubRelativeTarget(hub, lockPath)`.
  - The registration directory is `RunProcDir(filepath.Dir(BoltDir(hub)))`, which is `RunProcDir(hub)`, the path the test lists.
  - `closed` ends with no answer, so the error reads `…: EOF`. `error` ends with the `error …` line. `busy` returns `ErrPairBusy` with `try` and `ErrPairPushRunning` naming `rerun` without it.
  - Non-held rows record nothing.
  - `t.Cleanup(hold.finish(nil))` releases the stand-in's stdin and waits for it, so nothing is left running.
- **`failedRemoteSteps` and `noticeRemoteTeardown`.**
  - Removal with a weft tip and an empty `weftRoot` gives only the prefixed re-run, with no `git -C`. With `/weft` it gives the tag push, then the prefixed re-run.
  - `weftRootErr` set with `err` nil gives a single "lookup of its records repository" failure and no archive clause. The two are mutually exclusive, because an archive error returns before `weftRootErr` is set.
  - `RunTeardownChild` looks up the root only when `out.err != nil`, which is the only case that uses it. The `ResolveWorktree` failure passes an empty root.
  - The signature change has no caller outside `RunTeardownChild` and the tests. Vet and the tier-1 gate pass on both packages.
- **`NoticeRemoteTeardownEnded`.**
  - The `childHold.finish` write-failure path and the CLI signal path both reach it.
  - `warnWhenEnded` has no other caller.
  - The notifier is installed in the child. `main.go:156` runs `InstallPushNotifier` in the root `PersistentPreRunE` whenever `testing.Testing()` is false, so the mail is sent.
- **`holdTeardown`.** Its new `os.Executable` error returns before any spawn, and the `err` it declares does not shadow the earlier `err`.
- **Test strength.**
  - Each of these fails with its defect in place: `TestSpawnTeardownHold` (answer protocol, lock recording, `RoleTeardown`), `TestChildHold_Finish_NoticesAnUnwritableJob` (the bare Warn that mailed nothing), the `TestNoticeRemoteTeardown` "no weft root" rows, `TestNoticeRemoteTeardownEnded`, and the unread-job discard assertion.
  - The existing parallel row `UnwritableJobStillHandsOff` now calls the notice with an empty hub path. It cannot race the global notifier, because parallel tests resume only after the serial ones finish.

**Not a finding: the `weftRootErr` assignment is untested.** `RecordsRepositoryLookupFailure` builds the outcome with `weftRootErr` set directly. Nothing would fail if `runRemoteTeardown` stored that error in `out.err` again, because triggering a real `RecordsRepoRoot` failure after a successful archive needs a hub fixture. Only the reporting side of the split is pinned, which is a coverage limit rather than a defect.

**Tool calls**
1. Read `cmd/lym/main.go` lines 140–169, to check that the push notifier is installed in the detached teardown child, which my context lacked.
