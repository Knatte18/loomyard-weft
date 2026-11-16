# Card 10 — cancel-finish-and-removal

**Create:**
- `plan:internal/battencli#finishCancel` -> `func finishCancel(location *lymcwd.Location, slug string, remove bool, now time.Time) error`
- `plan:internal/pairteardown#TestRemovePair_DropsACancelledRecord` -> `func TestRemovePair_DropsACancelledRecord(t *testing.T)`
- `plan:internal/battencli#TestCancelIntegration_IntentAndFinishReachOrigin` -> `func TestCancelIntegration_IntentAndFinishReachOrigin(t *testing.T)`
- `plan:internal/battencli#TestClearDoneRun_LeavesACancelledRun` -> `func TestClearDoneRun_LeavesACancelledRun(t *testing.T)`
- `plan:internal/battencli#TestBattenPreStep_RefusesACancelledRun` -> `func TestBattenPreStep_RefusesACancelledRun(t *testing.T)`

**Edit:**
- `internal/battencli#cancelDeps`
- `internal/battencli#cancelReport`
- `internal/battencli#clearRunPhase`
- `internal/battencli#cancelNotice`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#cancelEnvelope`
- `internal/battencli#battenCLI.cancelCmd`
- `internal/battencli#cleanupDeps`
- `internal/battencli#fabricCleanupDeps`
- `internal/battencli#judgeRunDirectory`
- `internal/battencli#judgeStatus` -> `func judgeStatus(data []byte, prefix, slug string, pairStands bool) (candidate, tracked bool, reason string)`
- `internal/battencli#clearDoneRun`
- `internal/battencli#battenCLI.battenPreStepUnmarked`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown#BattenFollowUp`
- `internal/battencli#cancelFake`
- `internal/battencli#cancelFake.deps`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestCleanup`
- `internal/battencli#TestCancelIntegration_KeptPairJudgeOnly`
- `internal/battencli#TestCancelIntegration_RemoveTearsDownPairAndBranch`
- `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`
- `internal/pairteardown_test#TestBattenFollowUp`

**Uses:**
- `internal/pairteardown/doc.go`
- `plan:internal/shedrun#UpdateRunRecord`
- `plan:internal/shedengine#StateCancelled`
- `plan:internal/shedengine#CancelFinish`
- `plan:internal/pairteardown#PublishBattenRecord`
- `plan:internal/pairteardown#ReadRun`
- `internal/battencli#removeRunRecords`
- `internal/fabricengine#SpawnStatusPush`

**ImpactSummary:** a finished cancel keeps the batten record as `cancelled` in place of removing it, and every pair removal deletes a cancelled run's record with the pair.

**Intent:**
Finish a cancel into the batten record, and delete a cancelled run's record in the same step as every removal of its pair (D3).

`finishCancel` writes the finish through `shedrun.UpdateRunRecord`: its time and `Remove`, and the shed state `cancelled`, never `done`.
So the dobby's done-run clear, which deletes the task branch on origin, never takes a cancelled run.
`cancelDeps` gains `FinishCancel func(remove bool) (pushed bool, err error)`.
`hubCancelDeps` fills it with `finishCancel` followed by `pairteardown.PublishBattenRecord` with push.
`clearRunPhase` writes and publishes the finish first.
Without `--remove` it keeps the record: it removes no records, clears the claim, lifts the slug's pauses and posts the notice, and the run reads cancelled with keep.
With `--remove` it then removes the pair and branch and deletes the record through `RemoveRecords`, whose commit the existing push carries.
A crash between the pair's removal and the record's deletion leaves a cancelled record whose task branch is gone, which reads queued once the task is released.
`cancelReport` and `cancelEnvelope` gain `finished`; `records_removed` holds only under `--remove`.
`cancelNotice` and the `cancelCmd` help word a kept cancel as a cancelled record whose pair `lym batten cancel <slug> --remove` removes.

`Teardown` gains `dropCancelledRecord`, which `New` wires: when the slug's batten record reads cancelled, it removes the run directory, commits that removal on the prime's records side and spawns the status push.
`Teardown.RemovePair` calls it after a successful removal.
So `lym fabric remove <slug>` deletes the record in the same step, and so does the dobby teardown a later part adds; a failed drop warns and leaves the record to `lym batten cleanup`.
`BattenFollowUp` words a cancelled record that outlived its pair's removal with `lym batten cleanup --apply`, which removes it.

`lym batten cleanup` only repairs a record a crash left.
`cleanupDeps` gains `PairStands func(slug string) (bool, error)`, which `fabricCleanupDeps` reads from the pair's parts.
`judgeRunDirectory` passes it to `judgeStatus`.
`judgeStatus` makes a cancelled record a candidate when its pair is gone.
When the pair stands, it skips the record with the reason naming `lym fabric remove <slug>` as the way forward.
`clearDoneRun` reads the done state itself in place of calling `judgeStatus`, so a done-run clear leaves a cancelled record and its task branch.
`battenPreStepUnmarked` refuses a cancelled record: the run was cancelled, and the way forward is `lym batten cancel <slug> --remove`, then the release and the run again.

`TestRemovePair_DropsACancelledRecord` (tier 0, the teardown's fakes) asserts a cancelled record is dropped after the removal and that a running one is not.
`TestCancelIntegration_IntentAndFinishReachOrigin` (tier 1, hubforge) cancels with keep and asserts the record holds the intent and its finish on origin.
It also asserts, through `pairteardown.ReadRun` with the pair removed locally, that the run reads cancelled with keep.
A cancel interrupted after its intent was published reads cancelling the same way.
`TestCancelIntegration_RemoveTearsDownPairAndBranch` asserts `--remove` deletes the record with the pair, and that the released task's start refusal reads empty, so its dobby starts it.
`TestClearDoneRun_LeavesACancelledRun` (tier 0) asserts the clear leaves the record and the task branch.
`TestCleanup` gains the cancelled rows: removed once its pair is gone, refused naming `lym fabric remove <slug>` while it stands.
`TestBattenPreStep_RefusesACancelledRun` (tier 0) reaches the step refusal over a cancelled status file.

**Commit:** 10: a finished cancel keeps its record as cancelled, and a pair's removal deletes it
