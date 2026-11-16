# Card 6 — cancel-intent-replaces-marker

**Create:**
- `plan:internal/runfsm#ReadIntents` -> `func ReadIntents(prime *lymcwd.Location, slug string) (Intents, bool, error)`
- `plan:internal/runfsm#UpdateIntents` -> `func UpdateIntents(prime *lymcwd.Location, slug string, mutate func(*Intents) error) error`
- `plan:internal/runfsm#PublishIntents` -> `func PublishIntents(prime *lymcwd.Location, slug, msg string) (pushed bool, err error)`
- `plan:internal/runfsm#TestUpdateIntents_KeepsTheShedKeysAndClearsAnIntent` -> `func TestUpdateIntents_KeepsTheShedKeysAndClearsAnIntent(t *testing.T)`
- `plan:internal/battencli#TestCancelIntegration_AnInterruptedCancelReadsCancellingOnAnotherClone` -> `func TestCancelIntegration_AnInterruptedCancelReadsCancellingOnAnotherClone(t *testing.T)`

**Delete:**
- `internal/shedrun#CancelMarker`
- `internal/battencli#cancelMarker`

**Edit:**
- `internal/battencli#cancelMarkerReading`
- `internal/battencli#readCancelMarker`
- `internal/battencli#takeCancelMarker`
- `internal/battencli#heldCancelMarker`
- `internal/battencli#heldCancelMarker.release`
- `internal/battencli#cancelMarkerRefusal`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#endRunPhase`
- `internal/battencli#DobbyRuns`
- `internal/pairteardown#New`
- `internal/shedrun/doc.go`
- `internal/runfsm/doc.go`
- `internal/shedrun#TestPathConstructors`
- `internal/battencli#TestCancelMarker`
- `internal/battencli#TestArmSeed_CancelMarkerRefusesTheWrite`
- `internal/battencli#TestPreDriveCheck`
- `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`
- `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`
- `internal/battencli#newPreDriveReceiver`
- `internal/battencli#cancelFake.deps`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestRefuseStartOwnedElsewhere`
- `internal/battencli#TestCancelRun`

**Uses:**
- `internal/shedrun#CancelMarkerLock`
- `internal/shedrun#StatusFile`
- `internal/shedrun#StatusLock`
- `internal/shedrun#StatusRel`
- `internal/fabricengine#CommitAnchoredPaths`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#StatusPushLockWait`
- `internal/battencli#judgeCancelMarker`
- `internal/battencli#TestJudgeCancelMarker`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#battenCLI.refuseCancelMarker`

**Intent:**
The cancel intent in the batten record replaces the cancel marker of `internal/battencli/cancelmarker.go`; the cancel's lock stays local (D3).

runfsm gains the intent writers, under shedrun's tracked status path:
`ReadIntents` reads the `cancel_intent` and `pause_intent` keys of the prime's `shedrun.StatusFile(prime, slug)`, false when the file is absent.
`UpdateIntents` applies mutate under `shedrun.StatusLock` through `state.UpdateJSON` with a type that declares only those two keys, so every shed-owned key passes through, and Shed's own persists, which also go through `state.UpdateJSON`, keep the intents (PATTERN-tolerant-records); a cleared intent is omitted, so its key goes.
It writes nothing when the status file is absent: a slug with no batten record has no run to record a cancel or a pause in.
`PublishIntents` commits `shedrun.StatusRel` on the prime's records side with `fabricengine.CommitAnchoredPaths` and pushes it at once with `fabricengine.PushPairAnchored` and `fabricengine.StatusPushLockWait`, today's synchronous records push.
pushed is false, with a nil error, when origin is unreachable, and the push's own failed-push record and notice stand (PATTERN-pair-remote).

`takeCancelMarker` keeps taking `shedrun.CancelMarkerLock` exclusively first, then writes `CancelIntent{StartedAt, Machine, Remove}` (this machine's name, the `--remove` flag) and publishes it before `endRunPhase` ends anything; a re-run keeps an unfinished intent's `StartedAt`.
An unpublished intent stays committed here and the cancel goes on end-only as today, its output naming `lym batten cancel <slug>` once origin answers (D3's offline bound).
`readCancelMarker` reads the intent: present is an intent without a finish, or the lock held elsewhere; writer alive is the lock held, judged by `judgeCancelMarker` as today.
Taking the lock before the intent is written keeps a cancel visible to this machine's driving verbs in the window before its intent is committed, which answers the carried-over `cancel-invisible-before-intent-committed` finding: every driving verb on this machine (`battenCLI.refuseCancelMarker`, `startGuard.refuse`) already reads the lock; other machines read `cancelling` once the intent is pushed.
`cancelMarkerReading` carries the intent in place of the pid record; `cancelMarkerRefusal` names the machine and start time in place of a pid, and keeps both ways forward.
`heldCancelMarker.release` only drops the lock: there is no marker file left to remove, and the record keeps the intent.
`DobbyRuns` sets `RunView.CancelMarker` from the intent and the lock, and `pairteardown.New` reads the cancel for `registeredBattenAlive` the same way, through `runfsm.ReadIntents` at the prime.
`shedrun.CancelMarker` and the `cancelMarker` record go, with the doc paragraph that names the marker.
Until card 7 the cancel still removes the run's records at its end, so a finished cancel leaves no intent behind.

Grep `internal/` for `cancelMarker{` and `shedrun.CancelMarker(` before deleting, and move each hit, test fakes included, to the intent reading.
Tests: `TestUpdateIntents_KeepsTheShedKeysAndClearsAnIntent` (tier0) writes an intent over a status file holding shed keys and an unknown key, keeps all three, and clears it; `TestCancelMarker` and the other edited tests read the intent and the lock in place of the marker file;
`TestCancelIntegration_AnInterruptedCancelReadsCancellingOnAnotherClone` (tier1) stops a cancel after its intent is published and reads `cancelling` through `runfsm.Read` on a second clone of the hub, and the cancel's intent is on origin when `lym batten cancel` returns.

**ImpactSummary:** the cancel marker file goes; the cancel's start is recorded and pushed in the batten record, and every reader of the marker reads the intent and the local lock.

**Commit:** 6: A cancel records and pushes its intent in the batten record in place of the marker
