# Card 8 — cancel-readers-intent

**Edit:**
- `internal/battencli#cancelMarkerReading`
- `internal/battencli#judgeCancelMarker`
- `internal/battencli#readCancelMarker`
- `internal/battencli#heldCancelMarker`
- `internal/battencli#heldCancelMarker.release` -> `func (h heldCancelMarker) release() error`
- `internal/battencli#takeCancelMarker`
- `internal/battencli#cancelMarkerRefusal`
- `internal/battencli#cancelHold`
- `internal/battencli#cancelHold.finishMarker` -> `func (h *cancelHold) finishMarker() error`
- `internal/battencli#cancelHold.end`
- `internal/battencli#cancelRun`
- `internal/battencli#refuseBeamCancel`
- `internal/battencli#clearRunPhase`
- `internal/battencli#cancelDeps`
- `internal/battencli#endRunPhase`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#DobbyRuns`
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#battenCLI.refuseCancelMarker`
- `internal/battencli#battenCLI.preDriveCheck`
- `internal/battencli/cli.go`
- `internal/battencli#TestCancelMarker`
- `internal/battencli#TestJudgeCancelMarker`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestPreDriveCheck`
- `internal/battencli#newPreDriveReceiver`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestRefuseStartOwnedElsewhere`
- `internal/battencli#TestArmSeed_CancelMarkerRefusesTheWrite`
- `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`
- `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`
- `internal/beamcli#TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitAndTheEndedEntryAndPair`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.registeredBattenAlive`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestRegisteredBattenAlive_ReadsADobbyDrivenRun`
- `internal/pairteardown/doc.go`
- `internal/shedrun#CancelMarkerLock`
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun/doc.go`

**Delete:**
- `internal/battencli#cancelMarker`
- `internal/battencli#cancelMarkerStateLock`
- `internal/shedrun#CancelMarker`

**Uses:**
- `internal/shedrun#StatusFile`
- `internal/shedrun#StatusLock`

**Intent:** The cancel intent in the batten record replaces the cancel marker of `cancelmarker.go` as what every reader sees; the cancel's lock stays local (D3).
- `readCancelMarker` reads the batten record under its status lock: present is a cancel intent without a finish, and the writer is alive while the cancel lock, `shedrun.CancelMarkerLock`, is held.
  A cancel lock another process holds reads present with a live writer whether or not the record holds an intent yet:
  card 7 writes the intent only after the lock is taken and the Windows live-process check passes, and today's marker is written within the lock take,
  so without this a start guard, a pre-drive check or `DobbyRuns` would read no cancel in that window.
  So `readCancelMarker` tries the shared lock whenever the lock file exists, not only once an intent is seen.
  An absent lock file, its scratch root included, reads free with no cancel under way, and the read creates nothing, so a fresh machine or a removed `.lym` refuses nothing (D3).
  `judgeCancelMarker` takes that reading: a lock held elsewhere is present and live; with the lock free, present needs the intent both before and under the shared lock, as today's two marker stats do.
  `cancelMarkerReading.Marker` becomes `Intent shedengine.CancelIntent`.
  A record that does not decode reads present, so a torn write never hides a cancel.
- `takeCancelMarker` takes only the lock and writes no file; the intent is card 7's write.
  `heldCancelMarker.release` and `cancelHold.finishMarker` lose their remove flag, and `cancelDeps.TakeMarker` returns `release func() error` and no fresh flag, since there is no file to keep or remove.
  `cancelRun` and `endRunPhase` call them without it.
- `cancelMarkerRefusal` names, for a live writer, the cancel under way on this machine since the intent's `StartedAt`, with the way forward to wait for it to end and run the verb again; the pid lines go, since the lock, not a pid, tells liveness.
  A live writer whose intent is not written yet is named without a start time.
  For an interrupted one it names the intent's `Machine` and `lym batten cancel <slug>` run from that machine's prime to finish it, with `--remove` when the intent's `Remove` is set;
  run on another machine, that cancel refuses naming the adopt that takes the run over first, so the operator takes either path.
- `DobbyRuns` sets `RunView.CancelMarker` through `readCancelMarker`, so an unfinished cancel intent or a cancel lock another process holds reads as a cancel, not a file.
- `pairteardown`'s `cancelMarked` field becomes `cancelling`, read in `New` as an unfinished cancel intent in the batten record through `ReadBattenRecord`, or a cancel lock another process holds, its absent lock file reading free as above;
  `registeredBattenAlive` reads it, and the test fake `fakeSubstrates.teardown` sets it.
- `shedrun.CancelMarker` and the marker file go; `CancelMarkerLock`'s doc names it the cancel's local lock.
- The comments and probe texts that name the cancel marker name the record's cancel intent: `cli.go`'s package doc, the `startGuard` field doc, `hubStartGuard`'s doc, `startGuard.refuse`'s doc and its probe text, `refuseCancelMarker`'s probe text, `preDriveCheck`'s doc, `shedrun/doc.go`'s marker paragraph and `pairteardown/doc.go`'s liveness paragraph.
  So do `refuseBeamCancel`'s doc, which then refuses before the cancel's lock is taken, `clearRunPhase`'s doc, which leaves the cancel's lock and the pause gate with the caller, and `registeredBattenAlive`'s doc.

A run cancelled on another machine reads `cancelling` here once its record is pulled, and the start guard and the pre-drive check refuse it as an interrupted cancel naming that machine.

Tests: every listed test moves from writing or faking the marker file to writing an intent into the record or faking the reading with `Intent`.
`TestCancelMarker` (tier0) keeps its rows over the record: absent, intent with a free lock (interrupted), intent with a held lock (live), a finished intent (absent), an undecodable record (present),
and gains a row for an intent another machine wrote with `Remove` set, whose refusal names that machine and `--remove`,
and a row for a held lock with no intent in the record, reading present with a live writer and refused with no start time,
and a row for a prime with no scratch root, reading absent with no error and no lock file created.
`TestJudgeCancelMarker` (tier0) gains the held-lock-with-no-intent reading as present and live.
`TestRegisteredBattenAlive_ReadsADobbyDrivenRun` gains a row for a running record with an unfinished intent reading not alive.
`TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitAndTheEndedEntryAndPair` (tier1) gains a run with an unfinished intent in its record, read with `CancelMarker` set.
`TestPathConstructors` drops the marker row.

**ImpactSummary:** Every cancel-marker reader in battencli and pairteardown switches to the record's intent, the marker file and its constructor go; the refusal texts keep their ways forward and the comments follow.
