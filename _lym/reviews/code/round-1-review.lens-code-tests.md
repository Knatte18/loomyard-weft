```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

1. `RunTeardownChild` holds the lock until its input closes: pinned by `TestRunTeardownChild_HoldsTheLockUntilItsInputCloses`. It runs with no job only.
2. The child answers busy on a held lock, for both `try` and the bounded wait: pinned by `TestRunTeardownChild_AnswersBusyOnAHeldLock`.
3. `teardownChildArgs`: pinned by `TestTeardownChildArgs`.
4. The child reads a job and runs `runRemoteTeardown` in the order weft-tip push, archive tag, leased weft delete, task-branch delete: none. Every test that reaches `RunTeardownChild` or `holdTeardown` runs under `testing.Testing()`, so the in-process hold runs the job. The only child test feeds no job or a busy lock.
5. `childHold.finish` and `spawnTeardownHold` (lock handoff, held/busy/error answers, spawn failure, the lock file recorded in `rec`, the job written to stdin, a nil job closing stdin and waiting): none. `holdTeardown` takes the in-process path under `testing.Testing()`, so the detached path has no test.
6. The child holds the lock until the last remote step ends, so no push, Add or other teardown interleaves: none. Only the empty-job case is tested, with no remote step running while the lock is held.
7. Early-step failure skips the later steps: `runRemoteTeardown` returns when `archiveWeftTip` errors, so the weft delete and the task delete are skipped. I found no test row for it. Commit `351b4d18` changed the existing remove tests to assert "local teardown done, tip kept under the local tag", which may pin the local half. I did not read those tests.
8. `noticeRemoteTeardown` (the log, and the creel notice as one line naming slug, operation and way forward): none. A grep of the test files finds no `noticeRemoteTeardown` and no `installedPushNotifier` reference. The three failure branches, the discard way forward and the weft-delete failure each have no row. The way-forward text is a refusal-style message under `PATTERN-refusal-way-forward`.
9. `RunTeardownChild` resolve-worktree failure notice and the malformed-job error: none.
10. The `remote-teardown` verb in `fabriccli` (`teardownChildCmd`, the stdout signal, the `SetExit` paths): the roleguard row and the arity row only. The wiring has no test.
11. `RemoveResult.RemoteHandedOff` and the `remote_handed_off` envelope field (`removeFields`): none. Under `testing.Testing()` it is never true, and `removefields_test.go` changed 16 lines, but I did not confirm it sets the field.
12. `DiscardTaskBranch` and `cleanupOrphan` handoff paths (`job == nil` releases the lock, a skipped archive hands no job, `out.ran` false returns zero remote fields): no test for the handed-off branch. The in-process branch is pinned by the reworked `taskbranchdiscard_integration_test.go` and `pairbranch_archive_integration_test.go`.
13. `PushCommit` and `pushPlainWithRetry(remote, source, branch)`, used to push the weft tip with no local branch: no direct test row found. `push_test.go` and `pushbranch_integration_test.go` only lost lines. The in-process archive path may cover it through `Remove`, with an origin lacking the tip.
14. `fabric add` parallel `ls-remote` (`probeWarp`, `judgeWarpLeftover`, the `warpRead` WaitGroup): no new row found. The error ordering is changed: the weft error is returned before the warp error, and a warp read error is only checked after the weft judgement. Existing add-leftover tests may cover the sync results, but nothing pins the parallelism.
15. Local-only removal, verified by deleted tests: the deleted `*_local_integration_test.go` files and the dropped rows are consistent with the removed code.
16. `remove_localmode_integration_test.go` is unchanged in substance, 19 lines changed. It still covers `RemoveLocal` and `SnapshotUncommitted`, which the decision record says stay. Its name and header still say "local mode", which now means the no-origin removal. It is not a coverage gap, but `RemoveLocal` coverage overlaps the other remove tests only for the housekeeping and failed-push rows.
    - `APlainRemoveRunsTheHousekeeping` and the `Remove` row of `BothModesClearTheFailedPushRecords` test plain `Remove`, not local mode. They duplicate any `Remove` housekeeping and failed-push coverage in the other remove tests, which I did not confirm.

**Findings**

- MEDIUM, design. The detached handover has no test: `spawnTeardownHold`, `childHold.finish`, and `RunTeardownChild` running a real job. The focus's lock hold until the last step, the order across the handover, and the lock released on a failed spawn or a failed job write are unpinned. Evidence: `remoteteardown.go:177-186` (`!handOff || testing.Testing()` bypass), `remoteteardown.go:214-286`, and `remoteteardown_test.go` (no job-carrying case).
- MEDIUM, design. `noticeRemoteTeardown` has no test row. The log and creel notice line, its three failure branches, the discard branch and the `installedPushNotifier` nil path are uncovered. The line names slug, operation and way forward, which the operator's decision asked for. Evidence: `remoteteardown.go:130-165`, and no test reference found by grep.
- LOW, design. A failing early step in `runRemoteTeardown` skips the later steps (no weft delete, no task-branch delete). I found no test row for this. Evidence: `remoteteardown.go:106-109`.
- LOW, design. The `fabric add` parallel probe change (`probeWarp`, `judgeWarpLeftover`) and its changed error order have no new test row. Evidence: `add.go` diff, around lines 343-480.
- LOW, design. `RemoveResult.RemoteHandedOff` and the `remote_handed_off` envelope field have no pinned test. Evidence: `remove.go:113-114` and `fabric.go` `removeFields`.
- LOW, design. `PushCommit` and the new `source` parameter of `pushPlainWithRetry` have no direct test row. Evidence: `push.go`, the diff hunk at lines 130-160.
- NIT, consistency. The header of `remove_localmode_integration_test.go` and the name `newLocalModePair` keep the "local mode" wording after the local-only pair mode is gone, though `RemoveLocal` stays. The file's `Remove` rows `APlainRemoveRunsTheHousekeeping` and the `Remove` case of `BothModesClearTheFailedPushRecordsOfThePair` test plain `Remove`, so they may duplicate the other remove tests. I did not confirm that.

**Tool calls**
1. A grep across `_test.go` files for tests touching the new teardown symbols (`noticeRemoteTeardown`, `RemoteHandedOff`, `spawnTeardownHold`, `childHold`, `holdTeardown`, `PushCommit`, `judgeWarpLeftover` and others). My context held no view of the other test files.
