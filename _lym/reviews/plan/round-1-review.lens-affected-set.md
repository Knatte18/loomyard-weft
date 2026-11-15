```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: for each output, key, string or behaviour a card changes, I listed what reads or asserts it and which cards cover that set.

**Per change: the set found and the cards that cover it**

- **Card 6: board run status words** (the claim becomes `starting · Worktree-Create`; loom writes `halted`, `landing` or `paused`; batten writes `starting` or `halted`).
  - Readers that decide on it parse only the producer half: `RunHolds`, `ProjectRecordHeld`, `IsRunStatus` and `loomcli/peers.go:92,207`. Nothing to cover there.
  - Tests that assert the written text:
    - `TestWire_CreateWorktreeClaimsTheBoardEntry` (`wire_seams_test.go:697,714`): covered by card 6.
    - `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed` (`commitstatus_test.go:134`): covered by card 6.
    - `TestNewCommitStatusSeam_BoardStatus` (`wiring_commitstatus_test.go:127`): covered by card 6.
    - `TestBattenIntegration_Rows` (`lifecycle_integration_test.go:1067-1068`) asserts `running · Worktree-Create` after a refused row: **missed**.
  - Text that names the old form:
    - `docs/overview.md:1338` and `internal/loomcli/doc.go:66`: covered by card 6 only as `Uses`; their meaning still holds.
    - `contracts/orchrole/orch-role.md:312` names `running · Worktree-Create` as a pre-loom row status: **missed**.
    - The `battencli/cli.go:119` comment quotes the `running · Worktree-Create` claim: **missed**.
    - The `boardengine/runlock.go:13,16` doc comments say the state half is "the shed state": **missed**.
  - Fixtures that only seed a status (`dobbyengine` starts and view tests, `cancel_integration_test` seeds, `runlock_test`, `peers_test`, `adopt_integration_test`) read the producer half and are unaffected.
- **Card 8: the cancel marker file and its reader.**
  - Every Go reference to `shedrun.CancelMarker`, `readCancelMarker` or `takeCancelMarker` is covered across cards 8 and 11: `cancelmarker.go`, `cancelcmd.go:251-253`, `startguard.go:87,457`, `arm.go:765`, `dobbyreads.go:187`, `pairteardown/teardown.go:196`, `cancelmarker_test`, `predrive_test` (`TestPreDriveCheck`), `startguard_integration_test:107` (`TestArmAt_StartGuardRefusesBeforeAnyWrite`), `cancel_integration_test:139` (`TestCancelIntegration_BlockedRunWithNoPair`), `paths_test:111,115` (`TestPathConstructors`).
  - Prose naming the marker:
    - `pairteardown/doc.go:154`: covered by card 8.
    - `internal/dobbyengine/doc.go:154,286,338,385` still describe the "cancel marker": **missed**. Card 11 edits that file only for its pass-planning paragraph, and card 8's comment list omits it.
    - The cancel verb's Long text, "leaves its marker" (`cancelcmd.go:67`): **missed**. Card 7 rewrites that Long text only to drop the records removal, and card 8 does not list `cancelCmd`.
- **Card 7: cancel keeps the record; `records_removed` semantics; `record_kept` added.**
  - Tests asserting `records_removed`, in `cancel_test.go` and `cancel_integration_test.go`: covered by card 7 (`TestCancelRun` and the three integration tests).
  - Text: `docs/overview.md:1597`, `RunLockedError.Error` (`runlock.go:89-96`), `cancelCmd` Long, `Command` Long: covered by card 7.
- **Card 1: the `cancelled` batten state.**
  - Exhaustive state switches over the batten record:
    - `battencli/arm.go:530-540` (`checkRunStatus`) and `arm.go:902-909` (`battenPreStepUnmarked`): covered by card 7.
    - `battencli/cleanup.go:542-551` (`judgeStatus`): covered by card 9.
    - `battencli/drive.go:164-169`, whose default errors on an unknown end state: not listed. It is reachable only if a driver ends after a cancel finished, which the cancel's batten locks rule out, so it is not a finding.
- **Card 15: the pull-request record path moves under `RunDir`.**
  - Readers go through the constructor, so the path follows: `pairteardown/pullrequest.go:59`, `loomcli/landingdeps.go:348-349,432`, `prcontinue.go:78`, `pawlwiring.go:175-176`, `watchfilechanges_test.go:132`, `pawlwiring_integration_test.go:88`.
  - The writers are covered by card 15. `paths_test:77,79` is covered by card 15 through `TestPathConstructors`.
- **Card 16: the attempts path moves from `ScratchDir` to `RecordsDir`.**
  - The callers in `parentalign.go`, `finalize.go`, `fixerrun.go` and `fixaccept.go` are the listed members.
  - Tests are covered through the listed fixtures and tests (`finalize_gate_test`, `fixerrun_test`, `parentalign_fixer_test`). `fixerattempts_test` and `fixaccept_test` pass their own directories and are unaffected.
- **Card 9: the `judgeStatus` signature changes.** Its callers `clearDoneRun` and `judgeRunDirectory` are both listed on card 9.

**Findings**

1. MEDIUM, `design`, card 6 (`06-board-display-from-fsm.md`). Card 6 changes the claim to `starting · Worktree-Create` but does not list `internal/battencli#TestBattenIntegration_Rows`. That tier1 test asserts the old claim at `internal/battencli/lifecycle_integration_test.go:1067-1068`, so it fails once card 6 lands.
2. LOW, `consistency`, card 6. Three texts still describe the old status form and no card lists them:
   - `contracts/orchrole/orch-role.md:312` names `running · Worktree-Create` as the pre-loom row status.
   - The `internal/battencli/cli.go:119` comment says the claim is written as `running · Worktree-Create`.
   - `internal/boardengine/runlock.go:13,16` (`RunStatus` and its separator) still call the state half "the shed state".
3. LOW, `consistency`, cards 8 and 11. Two texts still describe the marker file after card 8 replaces it with the record's cancel intent:
   - `internal/dobbyengine/doc.go:154,286,338,385` keep saying "cancel marker". Card 11 edits that `doc.go` only for its pass-planning paragraph, and card 8's comment list omits it.
   - The `lym batten cancel` Long text, "leaves its marker" (`internal/battencli/cancelcmd.go:67`), is not named by card 7 or card 8.

Tool calls:
1. Grep for references to `CancelMarker`, `PullRequestRecord` and `FixerAttemptsPath`, and for files carrying state-word board statuses. The manifest held the definitions only, not their callers, tests or docs.
2. Grep for exact assertions of `running · Worktree-Create`, `blocked ·` and similar, plus the cancel integration test names. I needed which tests assert the words card 6 changes.
3. Awk for the enclosing test names at four assertion lines, and a grep of `loomcli/peers.go`. I needed the test names and whether peers parse the state half.
4. Grep for exhaustive `StateAwaiting` switches, docs saying cancel removes records or naming the cancel marker, and tests of `records_removed`. I needed the readers card 1's new state and card 7's text change reach.
5. Grep of the switch bodies in `drive.go`, `runreport.go`, `notice.go`, `arbiterstop.go` and `timingwaits.go`. I needed whether any of them refuses an unknown `cancelled` state.
6. Grep for docs defining the run status as `<state> · <producer>` or "shed state". I needed the prose card 6's display change falsifies.
