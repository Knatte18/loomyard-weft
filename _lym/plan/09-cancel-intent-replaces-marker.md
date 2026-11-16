# Card 9 — cancel-intent-replaces-marker

**Create:**
- `plan:internal/battencli#cancelReading` -> `type cancelReading struct`
- `plan:internal/battencli#readCancel` -> `func readCancel(location *lymcwd.Location, slug string) (cancelReading, error)`
- `plan:internal/battencli#heldCancel` -> `type heldCancel struct`
- `plan:internal/battencli#takeCancel` -> `func takeCancel(location *lymcwd.Location, slug string, wait time.Duration) (heldCancel, error)`
- `plan:internal/battencli#heldCancel.release` -> `func (h heldCancel) release() error`
- `plan:internal/battencli#cancelRefusal` -> `func cancelRefusal(slug string, reading cancelReading) error`
- `plan:internal/battencli#recordCancelIntent` -> `func recordCancelIntent(location *lymcwd.Location, slug string, remove bool, now time.Time) (bool, error)`
- `plan:internal/battencli#cancelHold.releaseCancel` -> `func (h *cancelHold) releaseCancel() error`
- `plan:internal/battencli#battenCLI.refuseCancelling` -> `func (c *battenCLI) refuseCancelling() error`
- `plan:internal/battencli#TestCancelIntentAndLock` -> `func TestCancelIntentAndLock(t *testing.T)`

**Delete:**
- `internal/battencli#cancelMarker`
- `internal/battencli#cancelMarkerReading`
- `internal/battencli#cancelMarkerStateLock`
- `internal/battencli#judgeCancelMarker`
- `internal/battencli#readCancelMarker`
- `internal/battencli#heldCancelMarker`
- `internal/battencli#heldCancelMarker.release`
- `internal/battencli#takeCancelMarker`
- `internal/battencli#cancelMarkerRefusal`
- `internal/battencli#cancelHold.finishMarker`
- `internal/battencli#battenCLI.refuseCancelMarker`
- `internal/battencli#TestCancelMarker`
- `internal/battencli#TestJudgeCancelMarker`
- `internal/shedrun#CancelMarker`
- `internal/shedrun#CancelMarkerLock`

**Edit:**
- `internal/battencli#cancelDeps`
- `internal/battencli#cancelReport`
- `internal/battencli#cancelHold`
- `internal/battencli#cancelHold.end`
- `internal/battencli#cancelRun`
- `internal/battencli#endRunPhase`
- `internal/battencli#refuseNothingLeft` -> `func refuseNothingLeft(slug string, remove, endOnly bool, rerun string, reading cancelReading, parts fabricengine.PairParts, deps cancelDeps) error`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#cancelEnvelope`
- `internal/battencli#battenCLI.cancelCmd`
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#battenCLI.refuseStartOwnedElsewhere`
- `internal/battencli#battenCLI.armSeed`
- `internal/battencli#battenCLI.preDriveCheck`
- `internal/battencli#DobbyRuns`
- `internal/dobbyengine#RunView`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/shedrun#TestPathConstructors`
- `internal/battencli#cancelFake`
- `internal/battencli#newCancelFake`
- `internal/battencli#cancelFake.deps`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestArmSeed_CancelMarkerRefusesTheWrite`
- `internal/battencli#TestRefuseStartOwnedElsewhere`
- `internal/battencli#newPreDriveReceiver`
- `internal/battencli#TestPreDriveCheck`
- `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`
- `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`

**Uses:**
- `internal/dobbyengine/doc.go`
- `internal/pairteardown/doc.go`
- `internal/shedrun/doc.go`
- `plan:internal/shedrun#CancelLock`
- `plan:internal/shedrun#UpdateRunRecord`
- `plan:internal/shedengine#CancelIntent`
- `plan:internal/pairteardown#PublishBattenRecord`
- `internal/pairteardown#ReadBattenRecord`

**ImpactSummary:** the cancel marker file goes; a cancel's intent is a tracked field of the batten record, committed and pushed before the cancel ends anything, and every reader of the marker reads the intent.

**Intent:**
Replace the cancel marker of `internal/battencli/cancelmarker.go` with the cancel intent in the batten record; the cancel's lock stays local (D3).

`cancelReading` holds `Present` (the record holds a cancel intent without its finish), `WriterAlive` (a process holds `shedrun.CancelLock`) and the `Intent`.
`readCancel` reads the record tolerantly and probes the lock with a shared try, as the marker's reading did; a record that does not decode reads present, so a torn write never hides a cancel.
`takeCancel` takes the lock exclusively within the wait, creating the scratch root; a lock still held refuses through `cancelRefusal` as a live cancel.
`heldCancel.release` drops the lock.
`cancelRefusal` words a live cancel as under way since the intent's start on its machine, with the way forward to wait for it to end and then run the verb again.
It words an interrupted one with the way forward to run `lym batten cancel <slug>` from the prime to finish it.
No pid is named any more, since the intent records none.
`recordCancelIntent` writes the intent through `shedrun.UpdateRunRecord`: the start time, this machine's name and `Remove`.
An interrupted cancel's intent keeps its start and machine and takes the new `Remove`; a slug with no batten record records nothing and reports false.

`cancelDeps` trades `ReadMarker` and `TakeMarker` for `ReadCancel` and `TakeCancel`.
It gains `RecordIntent func() (bool, error)` and `PublishIntent func(push bool) (pushed bool, err error)`, which `hubCancelDeps` fills with `recordCancelIntent` and `pairteardown.PublishBattenRecord`.
`endRunPhase` takes the lock, then judges the Windows survivors, then records the intent and publishes it, all before `EndForCancel` ends anything.
With origin reached, a push that did not land refuses with the re-run as way forward, having ended nothing.
With origin unreachable, the intent is committed and the cancel goes on end-only as today, naming the re-run once origin answers.
So the intent is on origin before anything ends, and on this machine every reader sees it from the moment it is written, which answers the carry-over finding cancel-invisible-before-intent-committed.
`cancelHold` keeps the lock's release in place of the marker's, released once by `releaseCancel`; `cancelRun` releases it at the end.
`cancelReport` and `cancelEnvelope` gain `intent_pushed`; the `cancelCmd` help says the cancel records and pushes its intent first.
`refuseNothingLeft` counts a present intent as left.
The records removal of the clear phase is unchanged on this card, so a finished cancel still removes the record with its intent; the next card keeps it.

The marker's readers read the intent:
`startGuard`'s `Marker` field becomes `Cancel`, read by `startGuard.refuse` and by `refuseCancelling`, which replaces `refuseCancelMarker` at its three call sites.
`DobbyRuns` sets the run view's cancel flag from the record it already read, an intent without its finish, and `RunView`'s doc says so.
`New` reads the same from `ReadBattenRecord` for the teardown's `cancelMarked`, whose doc on `Teardown` says so.
`shedrun.CancelMarker` and `CancelMarkerLock` go, with their `TestPathConstructors` rows.

`TestCancelIntentAndLock` (tier 0, over a temporary location) walks an intent through recording, a live lock, a dead writer and a torn record.
It asserts each refusal's way forward.
`TestCancelRun`'s rows assert that `RecordIntent` and `PublishIntent` come before `EndForCancel`, that a push that did not land ends nothing, and that an end-only cancel commits without pushing.
The other listed tests swap the marker for an intent written into the status file.

**Commit:** 9: a cancel records and pushes its intent in the batten record before it ends anything
