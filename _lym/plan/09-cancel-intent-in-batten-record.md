# Card 9 — cancel-intent-in-batten-record

**Create:**
- `internal/runfsm/intent_test.go`
- `plan:internal/battencli#TestArmSeed_CancelIntentRefusesTheWrite` -> `func TestArmSeed_CancelIntentRefusesTheWrite(t *testing.T)`
- `plan:internal/runfsm#RecordCancelIntent` -> `func RecordCancelIntent(location *lymcwd.Location, slug string, intent shedengine.CancelIntent) (found bool, err error)`
- `plan:internal/runfsm#RecordCancelFinish` -> `func RecordCancelFinish(location *lymcwd.Location, slug string, finish shedengine.CancelFinish) error`
- `plan:internal/runfsm#PublishBattenRecord` -> `func PublishBattenRecord(location *lymcwd.Location, slug, msg string, push bool) error`
- `plan:internal/runfsm#TestRecordCancelIntent` -> `func TestRecordCancelIntent(t *testing.T)`
- `plan:internal/shedrun#CancelLock` -> `func CancelLock(l *lymcwd.Location, runID string) string`
- `plan:internal/battencli#cancelReading` -> `type cancelReading struct`
- `plan:internal/battencli#readCancel` -> `func readCancel(location *lymcwd.Location, slug string) (cancelReading, error)`
- `plan:internal/battencli#takeCancelLock` -> `func takeCancelLock(location *lymcwd.Location, slug string, wait time.Duration) (release func() error, err error)`
- `plan:internal/battencli#cancelRefusal` -> `func cancelRefusal(slug string, reading cancelReading) error`

**Edit:**
- `internal/battencli#cancelDeps`
- `internal/battencli#cancelReport`
- `internal/battencli#cancelHold`
- `internal/battencli#cancelHold.finishMarker`
- `internal/battencli#cancelHold.end`
- `internal/battencli#cancelRun`
- `internal/battencli#endRunPhase`
- `internal/battencli#clearRunPhase`
- `internal/battencli#refuseNothingLeft`
- `internal/battencli#cancelNotice`
- `internal/battencli#cancelEnvelope`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#battenCLI.refuseCancelMarker`
- `internal/battencli#battenCLI.armSeed`
- `internal/battencli#battenCLI.preDriveCheck`
- `internal/battencli#battenCLI.refuseStartOwnedElsewhere`
- `internal/battencli#DobbyRuns`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.registeredBattenAlive`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestCancelCmd_RefusesBeforeAnything`
- `internal/battencli#TestCancelIntegration_KeptPairJudgeOnly`
- `internal/battencli#TestCancelIntegration_RemoveTearsDownPairAndBranch`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestPreDriveCheck`
- `internal/battencli#TestRefuseStartOwnedElsewhere`
- `internal/pairteardown#TestRegisteredBattenAlive_ReadsADobbyDrivenRun`

**Delete:**
- `internal/battencli#TestArmSeed_CancelMarkerRefusesTheWrite`

**Extends:**
- `internal/runfsm/doc.go`
- `internal/shedrun/doc.go`
- `internal/pairteardown/doc.go`

**Uses:**
- `plan:internal/shedengine#CancelIntent`
- `plan:internal/shedengine#CancelFinish`

**Intent:** `lym batten cancel` records its intent and its finish in the batten record, committed and pushed, in place of the cancel marker (D3).
This answers the carried-over Plan-Review finding `cancel-invisible-before-intent-committed`: no machine ends anything before its intent is on origin, bar the offline case D3 bounds.

The runfsm writers resolve the prime from `location` as `Read` does and write the prime's `shedrun.StatusFile` of the slug under `shedrun.StatusLock` through `state.UpdateJSON`, so shed's persists and these writes merge.
`RecordCancelIntent` sets `CancelIntent` when the record exists and keeps a standing intent's `StartedAt` and `Machine`; `found` is false and nothing is written when there is no batten record.
`RecordCancelFinish` sets the intent's `Finish` and the state `cancelled`, never `done`, so `battencli.ClearDoneRun` never takes the run.
`PublishBattenRecord` commits the prime's run paths for the slug through `fabricengine.CommitAnchoredPaths`;
with `push` it then pushes through `fabricengine.PushPairAnchored` with `fabricengine.StatusPushLockWait` and returns the push's error.
`TestRecordCancelIntent` goes in `intent_test.go`:
it writes an intent and a finish over a seeded record on a temporary prime layout and asserts the decoded record, a standing intent kept, an absent record left absent, and an unknown key kept.
`intent_test.go` is tier0.

`shedrun.CancelLock` is the lock a live cancel holds, `<run segment>.cancel.lock` directly under `ScratchRoot`, the path `CancelMarkerLock` names today; the lock stays local.

In battencli the cancel's order becomes:
judge as today; read the cancel through `readCancel`, which reports the record's intent and whether a live cancel holds `CancelLock`, and refuse a live one through `cancelRefusal`;
take `CancelLock` through `takeCancelLock`;
`RecordCancelIntent` with this machine, the time and `--remove`, then `PublishBattenRecord`, pushing unless origin was found unreachable;
a push that fails while origin answered refuses before anything is ended, naming the rerun;
then end the processes and strands as today.
Offline (end-only), the intent is committed and not pushed, the processes end, and the way forward is the rerun once origin answers, which pushes the intent and finishes.
`clearRunPhase` then writes the finish with `RecordCancelFinish` and commits it.
Without `--remove` the record stays, with its finish, and is pushed: `RecordsRemoved` stays false.
With `--remove` it removes the pair and its task branch, then deletes the record with `RemoveRecords`, the same step of the cancel, so a crash between the two leaves a finished cancelled record whose task branch is gone, which D1 reads as no record.
The push of the finish or the removal runs synchronously at the end; a failed push warns and spawns the detached push.
The claim clear and the pause lifts stay as today.
`cancelReading` replaces `cancelMarkerReading` everywhere outside `cancelmarker.go`:
the start guard's seam, `refuseCancelMarker`, `armSeed`, `preDriveCheck` and `refuseStartOwnedElsewhere` refuse a run whose record carries an intent without a finish, naming `lym batten cancel <slug>` to finish it, or the wait for a live one;
`DobbyRuns` sets `RunView.CancelMarker` from an intent without a finish;
pairteardown's `cancelMarked` seam becomes `cancelling`, read from the record's intent without a finish.
`cancelEnvelope` gains `intent_pushed` and `record_kept`.
`cancelmarker.go` keeps compiling and loses its last callers here; card 10 deletes it.
`runfsm/doc.go`, `shedrun/doc.go` and `pairteardown/doc.go` each gain the sentences naming the intent, the lock and the seam.

Tests: `TestCancelRun` (tier0) asserts the order intent-recorded, published, then ended; a failed push refusing before any end; the offline intent committed unpushed; the keep finish published with the record kept; and the remove finish followed by pair removal and record deletion.
The two integration tests (tier1) assert the record on origin after a kept cancel, carrying its finish, and the record gone with the pair after `--remove`.
The guard, pre-drive and owner tests swap their marker fixtures for an intent in the record.
`TestArmSeed_CancelIntentRefusesTheWrite` (tier0) replaces `TestArmSeed_CancelMarkerRefusesTheWrite`, asserting the same refusal from an intent without a finish.

**ImpactSummary:** Rewires the whole cancel and every cancel-marker reader in battencli and pairteardown onto the batten record; the marker file is no longer read or written.

**Commit:** 9: cancel-intent-in-batten-record
