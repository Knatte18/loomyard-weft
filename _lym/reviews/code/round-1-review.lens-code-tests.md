```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for a test row that would fail if the behaviour broke. I worked from the test names and rows the diff adds, and read no test bodies.

**Behaviour → pinning test**
- `PushPairAnchored` sends records first, then code: `pushanchored_integration_test` ("UncoveredCallPushesTheRecordsSideBeforeTheCodeSide", `PushesBothSidesRetriesAndReportsEachSide`). Covered.
- A failed or rejected records side withholds the code side, and the error names the withheld side: "RefusingRecordsRemoteWithholdsTheCodeSide", `Deferred_*`. Covered.
- `coalescePushSides` with a flagged or lock-wait-gave-up code side: "ACodeFlagLeftByAGivenUpPairPushStaysUntilAPairPush", "CoveredCallDefersAtTheRecordsPush", `TestCoalescePushRequest_CoveredCallDefers`. Covered.
- Teardown wait turns the push owed, or releases inside the wait: "TeardownLockHeldPastTheWaitOwesTheRecordsPush" and "TeardownLockReleasedInsideTheWaitPushesBothSides". Covered.
- `IsPushOwed` classification: `TestIsPushOwed` (all classes, joined, nil, handed over). Covered.
- Rejected-push takeover read, covering:
  - another machine's entry;
  - a former name of this machine;
  - a hand-over mark naming the entry's machine, and a mark naming a different machine;
  - zero names;
  - no upstream;
  - a prime rejected by other runs' commits;
  - the anchor change's push.

  All in `TestRejectedRecordsPush_ReadsTheTakeover`. Covered.
- `HandOverTips`: archive tags, a parent ahead by the run's merge alone versus a commit beyond it, tip push under lease, a given-up tip reset, a refused tag push leaving no mark, the dropped-intent notice. `TestHandOverTips`, `TestArchiveParentMerge`. Covered.
- `pairteardown.PushRun`/`HandOver`, outside the pair, inside the pair, dropped intents, stale prime tracking ref: `TestPushRun_HandsOverOnATakeover`. Covered.
- Teardown-lock respawn retired: "FailedRemoveSpawnsNoPushAndReleasesTheLock" and "FailedRemoveAfterTheTaskTeardownSpawnsNoPush". Covered.
- Pending-flag respawn retired: "ForeignFlagStaysOnDisk". Covered.
- `ReadRun` `PushOwed`, `CodeTipMissing`, error and worktree (`readFailedPush`, `FailedPushesOwedBy`, runs union): `TestReadRun`, "OwedByListsTheRunsOwedClassesNewestFirst", "RunsRoundTripAndUnionAcrossFailuresOfOneBranch". Covered.
- `OriginCodeTip`: `TestOriginCodeTip`. Covered.
- `ReturnToPreviousTransition`, including skipping non-transition commits and a single transition returning false: `TestReturnToPreviousTransition`. Covered.
- `runfsm` act rule, refusal text and `Aside`: `TestAct` rows (owed, owed-bare, missing tip with and without machine, aside). Covered.
- `statuscommit`: push, owed retry with backoff, ceiling and a nil `Push` panic: `TestNew_*`. Covered.
- Admission before the commit and release on every exit: no test row seen. Gap (F1).
- Merge-active skip and a commit error still releasing the admission: no row visible. Included in F1.
- `loomcli` seam wiring (board status before the push, transition trailer names the code tip and the run, finish): `wiring_commitstatus_*`. Covered.
- Intent push (tender pause/resume, batten cancel): `TestOwedIntentPush`, `TestCancelRun`. Covered.
- `lym batten cancel` output naming the re-run `lym batten cancel <slug>`: only `TestCancelVerdict` and `TestCancelRun` are in the redundancy report; I read no bodies, so I cannot confirm the re-run wording is asserted. Possible gap (F2).
- Pull-request record and fixer attempt counts pushing synchronously: `TestPullRequestDeps_CommitsTheRecord` for the PR record. No test row visible for fixer attempt counts. Gap (F3).
- `Finalize` order (parent push before records before board done, owner reads, archive on a rejected parent push, a re-step pushing the parent first, an unreadable owner returning a transient error, a handed-over push): `finalize_test`, `finalize_owner_test`. Covered.
- `Board.SetStatusPushed`: `TestBoard_SetStatusPushed`. Covered.
- The run-claim and owner synchronous push is pre-existing and unchanged.
- A display-only board status write staying detached: no assertion seen. Gap (F4).
- `fabriccli` `weft_verbs.go` change (15 lines): only `doc.go` and `testmain_test.go` changed in tests, so no row pins the new behaviour. Gap (F5).
- `beamcli/deps.go` now passes `PairPush{LockWait: StatusPushLockWait}`: no test row. Gap (F6).
- `lym beam status` showing the owed push and last error: `beamcli` has no test change, but the shown state comes from `ReadRun`, which `TestReadRun` covers.
- Mutual-exclusion lens items I did not check: `TestIsPushOwed` already covers `ErrPairPushRunning`.

**Findings**
- F1 — MEDIUM, design. `internal/statuscommit/statuscommit.go:104-125`. The admission taken before the commit has no test: `Deps.Admit` is released on a commit error, a no-op commit, a landed push, an owed push and a failed push, and no `TestNew_*` row counts acquire and release. Add a row asserting exactly one release on each exit and that the first push runs before the release.
- F2 — LOW, design. `internal/battencli/cancel.go` (cancel output). I could not confirm that `TestCancelRun` asserts the output naming the re-run `lym batten cancel <slug>` when the push is owed. If it does not, add that assertion.
- F3 — MEDIUM, design. The fixer attempt counts' synchronous push (D1) has no test in this diff. Add a row proving a count's write is pushed before the next producer, or an owed push is returned.
- F4 — LOW, design. A display-only board status write staying on the detached board sync: no test asserts the split between a decision write and a display-only write.
- F5 — MEDIUM, design. `internal/fabriccli/weft_verbs.go:15 changed lines`. The changed verb behaviour has no new test row, only `doc.go` and `testmain_test.go` changed. Add a verb-level test for the changed behaviour.
- F6 — LOW, design. `internal/beamcli/deps.go`. The beam-up push now passes `PairPush{LockWait: StatusPushLockWait}`, which no test pins.

**Tool calls**
1. `git diff … -- '*_test.go'` filtered for test names and table rows: the manifest held no test file.
2. `git diff … --stat` and `-U0` on `beamcli`, `pullrequest`, `tenderengine`, `pauserecord`, `sheddriver` and `fabriccli`: to see which of these have tests.
