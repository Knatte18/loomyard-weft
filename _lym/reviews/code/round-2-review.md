---
verdict: APPROVED
---

No findings. The range reviewed is `git diff 143307f6..HEAD`; verdicts on the three focus questions follow.

## Focus 1: `SyncOptions.notifier` precedence and fallback

`TestSyncOptionsNotifier` (`internal/fabricengine/remoteteardown_test.go`) has two rows.
`NilNotifierFallsBackToTheInstalledOne` expects one mail to the installed notifier and none to an own notifier.
`OwnNotifierShadowsTheInstalledOne` expects none to the installed notifier and one to the own one.
Both precedence and fallback are pinned with exact counts.
The test and its rows install the process's push notifier through `recordNotices` and call no `t.Parallel`, so they run serial.
The other tests that install the notifier (`TestNoticeRemoteTeardown`, `TestNoticeRemoteTeardownEnded`, `TestRunTeardownChild_NoticesAnUnreadableJob`
and `TestRunTeardownChild_RunsTheRemoteStepsInOrderUnderTheLock`) are serial too.

## Focus 2: `NewTopology` default gc threshold

The `NewTopology starts at git's default gc.auto` subtest of `TestStoreHousekeeping` reads the threshold through `GCAutoThresholdForTest(NewTopology(Config{}))`.
It compares it to the literal 6700 and overrides nothing, so a zero value fails it.
It is not a duplicate: the other rows set the threshold to 1 on their own `Topology`.

## Focus 3: parallel tests against the installed notifier

After the diff no test sets `originCallHook`, `originVerdictHook`, `anchorFailHook`, `weftHeadSHA` or the old `gcAutoThreshold`; a grep finds no reference to any of them.
Every writer of `installedPushNotifier` in the package is a serial top-level test.
Parallel tests that still reach a path reading it (`RecordPushFailure` in `TestFailedPushRecord`'s later subtests, `TestReconcileOrigin_Ended`, the removal tests)
read it while it is nil: the Go runner finishes all serial top-level tests, cleanups included, before any parallel one resumes.
The variable stays because the detached teardown child has no options to carry a notifier; the change description gives that reason, and the diff bears it out.

## Other checks

- Limits hold: every changed file sits in `internal/fabricengine`, and the production paths read the same values, now from options, `Topology` and `anchorSeams` (zero value is production).
- `TestWorktreeSeam_Events` and `TestReconcile_MigratesMarkerlessWefts` stay serial with their causes named in the change description,
  as the board entry's Goal allows when removing the cause needs a larger change.
- No `Pattern-Departure:` trailer is present and no `PATTERN` entry is declared or changed.
- `TestStoreHousekeeping_Warnings` keeps every log assertion of the old test; the packing assertion moves to a loose-object count check without loosening it.
