# Card 10 — orch-cycle-over-creel

**Create:**
- `plan:internal/orchengine#PendingStep` -> `type PendingStep struct`
- `plan:internal/orchengine#stepInjectTTL` -> `const stepInjectTTL = 10 * time.Minute`
- `plan:internal/orchengine#cycleHoldDuration` -> `const cycleHoldDuration = 10 * time.Minute`
- `plan:internal/orchengine#Watcher.orchName` -> `func (w *Watcher) orchName(st State) (string, error)`
- `plan:internal/orchengine#Watcher.creelFailed` -> `func (w *Watcher) creelFailed(st *State, err error) error`
- `plan:internal/orchengine#Watcher.renewHold` -> `func (w *Watcher) renewHold(st *State) (bool, error)`
- `plan:internal/orchengine#Watcher.releaseHold` -> `func (w *Watcher) releaseHold(st *State) error`
- `plan:internal/orchengine#Watcher.sendStep` -> `func (w *Watcher) sendStep(st *State, mail bool, step CreelStep, body string, now time.Time) error`
- `plan:internal/orchengine#Watcher.adoptPending` -> `func (w *Watcher) adoptPending(st *State) (bool, error)`
- `plan:internal/orchengine#Watcher.stepOutcome` -> `func (w *Watcher) stepOutcome(st *State) (CreelOutcome, bool, error)`
- `plan:internal/orchengine#Watcher.readReplies` -> `func (w *Watcher) readReplies(messages []CreelMessage) error`
- `plan:internal/orchengine#Watcher.turnEndAfterSending` -> `func (w *Watcher) turnEndAfterSending(sendingAt time.Time) (shuttleengine.Event, bool)`
- `plan:internal/orchengine#Watcher.resetForOtherStrand` -> `func (w *Watcher) resetForOtherStrand(st State) (State, error)`
- `plan:internal/orchengine#ClearResumeNoticed` -> `func ClearResumeNoticed(recordPath, strand string, withdrawn []CreelWithdrawn, now time.Time) error`
- `plan:internal/orchengine#fakeSession.StrandColor` -> `func (f *fakeSession) StrandColor(guid string) (string, error)`
- `plan:internal/orchengine#TestWatcher_NoteReplyGate` -> `func TestWatcher_NoteReplyGate(t *testing.T)`
- `plan:internal/orchengine#TestWatcher_StepOutcomes` -> `func TestWatcher_StepOutcomes(t *testing.T)`
- `plan:internal/orchengine#TestWatcher_CycleHold` -> `func TestWatcher_CycleHold(t *testing.T)`
- `plan:internal/orchengine#TestWatcher_CreelFailuresHoldTheCycle` -> `func TestWatcher_CreelFailuresHoldTheCycle(t *testing.T)`
- `plan:internal/orchengine#TestWatcher_TurnEndCountsOnlyAfterSending` -> `func TestWatcher_TurnEndCountsOnlyAfterSending(t *testing.T)`
- `plan:internal/orchcli#runnerSession.StrandColor` -> `func (s runnerSession) StrandColor(guid string) (string, error)`

**Edit:**
- `internal/orchengine#Session`
- `internal/orchengine#State`
- `internal/orchengine#ResetForFreshLaunch`
- `internal/orchengine#Watcher`
- `internal/orchengine#phaseEvents`
- `internal/orchengine#Watcher.Tick`
- `internal/orchengine#Watcher.tick`
- `internal/orchengine#Watcher.Run`
- `internal/orchengine#Watcher.recordSessionID`
- `internal/orchengine#Watcher.typeStartColor`
- `internal/orchengine#Watcher.typeColor` -> `func (w *Watcher) typeColor(st *State) error`
- `internal/orchengine#Watcher.initCursor`
- `internal/orchengine#Watcher.toIdle`
- `internal/orchengine#Watcher.enter`
- `internal/orchengine#Watcher.confirm`
- `internal/orchengine#Watcher.markStuck`
- `internal/orchengine#Watcher.logTyped`
- `internal/orchengine#Watcher.tickIdle`
- `internal/orchengine#Watcher.renderHandoffRequest`
- `internal/orchengine#Watcher.tickHandoff`
- `internal/orchengine#Watcher.startClearing`
- `internal/orchengine#Watcher.tickClearing`
- `internal/orchengine#Watcher.startAutoReload`
- `internal/orchengine#Watcher.startReload`
- `internal/orchengine#Watcher.typeReloadStep`
- `internal/orchengine#Watcher.advanceReload`
- `internal/orchengine#Watcher.settleSkillTurn`
- `internal/orchengine#Watcher.tickResuming`
- `internal/orchengine#Watcher.startCompacting`
- `internal/orchengine#Watcher.reloadAfterCompaction`
- `internal/orchengine#Watcher.tickCompacting`
- `internal/orchengine#Watcher.queueResumeLine`
- `internal/orchengine#RenderHandoffInstruction`
- `internal/orchengine#RenderSoftHandoffInstruction`
- `internal/orchengine#render`
- `internal/orchengine#fakeSession`
- `internal/orchengine#fakeSession.count`
- `internal/orchengine#fakeCreel.Inject`
- `internal/orchengine#watchEnv`
- `internal/orchengine#newWatchEnv`
- `internal/orchengine#watchEnv.injectHandoff`
- `internal/orchengine#watchEnv.writeHandoff`
- `internal/orchengine#watchEnv.reachClearing`
- `internal/orchengine#watchEnv.reachResuming`
- `internal/orchengine#watchEnv.assertNoCalls`
- `internal/orchengine#watchEnv.withSkills`
- `internal/orchengine#watchEnv.callsAfter`
- `internal/orchengine#watchEnv.endTurn`
- `internal/orchengine#watchEnv.assertReload`
- `internal/orchengine#newSoftEnv`
- `internal/orchengine#watchEnv.injectSoft`
- `internal/orchengine#watchEnv.reloadStepOf`
- `internal/orchengine#newCompactEnv`
- `internal/orchengine#watchEnv.compactFocus`
- `internal/orchengine#watchEnv.finishNote`
- `internal/orchengine#watchEnv.reachCompacting`
- `internal/orchengine#watchEnv.landBoundary`
- `internal/orchengine#watchEnv.landBoundaryFollowedBy`
- `internal/orchengine#watchEnv.assertCompactCalls`
- `internal/orchengine#watchEnv.callCount`
- `internal/orchengine#reloadPluginsCall`
- `internal/orchengine#reloadSkillsCall`
- `internal/orchengine#TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`
- `internal/orchengine#TestRenderPointers_OneLineNamingTheirPaths`
- `internal/orchengine#TestRenderPointers_MultiLineOverrideFails`
- `internal/orchengine#TestSaveLoadState_RoundTrip`
- `internal/orchengine#TestResetForFreshLaunch`
- `internal/orchengine#TestCompact_CompletesOnBoundaryAfterEntryAndIdle`
- `internal/orchengine#TestCompact_CompletionRemovesOnlyASatisfiedCompactRequest`
- `internal/orchengine#TestCompact_CycleReloadsPluginsThenPointerNamingTheNote`
- `internal/orchengine#TestCompact_DeferTurnEndChangesNothing`
- `internal/orchengine#TestCompact_DeferTurnEndWhileCompactingChangesNothing`
- `internal/orchengine#TestCompact_PersistsCompactingBeforeTyping`
- `internal/orchengine#TestCompact_RenderFailureAfterNoteChangesNothing`
- `internal/orchengine#TestCompact_RequestedCycleClearsRequestAndCompactsAfterNote`
- `internal/orchengine#TestCompact_RestartRetypesOnlyOnIdleWithNoBoundary`
- `internal/orchengine#TestCompact_RestartWithBoundaryReadTypesNothing`
- `internal/orchengine#TestCompact_SoftCycleHonoursDefer`
- `internal/orchengine#TestCompact_TimeoutDoesNotHoldRequestedCycle`
- `internal/orchengine#TestCompact_TimeoutHoldsSoftTrigger`
- `internal/orchengine#TestCompact_TimeoutReturnsToIdleAndHoldsAutomaticTriggers`
- `internal/orchengine#TestCycleRequest_AbandonedWithinTimeoutOfRequest`
- `internal/orchengine#TestCycleRequest_ModeFollowsRequestNotConfig`
- `internal/orchengine#TestCycleRequest_RestartedWatcherMeasuresFromRequestTime`
- `internal/orchengine#TestCycleRequest_StaleMarkerRemovedAtFirstTick`
- `internal/orchengine#TestWatcher_AbortedHandoffNoRetryUntilLaterTurnEnd`
- `internal/orchengine#TestWatcher_AboveThresholdInjectsHandoffPersistedFirst`
- `internal/orchengine#TestWatcher_AskDuringHandoffAborts`
- `internal/orchengine#TestWatcher_AskEventBlocksInjection`
- `internal/orchengine#TestWatcher_AutoCompactionBoundaryAtOrBeforeBaselineTriggersNothing`
- `internal/orchengine#TestWatcher_AutoCompactionReloadGoesIdleWhenTheIndexSourceFails`
- `internal/orchengine#TestWatcher_AutoCompactionReloadsOnlyAFreshBoundary`
- `internal/orchengine#TestWatcher_AutoCompactionReloadsPluginsThenPointer`
- `internal/orchengine#TestWatcher_BelowThresholdNoInjection`
- `internal/orchengine#TestWatcher_ClearCycleReloadsSkillsThenPointer`
- `internal/orchengine#TestWatcher_ClearingTimeoutNeverTypesWhileBusy`
- `internal/orchengine#TestWatcher_ClearRendersRoleFileBeforeResume`
- `internal/orchengine#TestWatcher_ConfirmedClearRemovesOnlyASatisfiedClearRequest`
- `internal/orchengine#TestWatcher_DeferIgnoredInHardAndRequestedCycles`
- `internal/orchengine#TestWatcher_EveryAbortReturnPersistsCursor`
- `internal/orchengine#TestWatcher_FailingHandoffRenderChangesNothing`
- `internal/orchengine#TestWatcher_FailingRenderAbortsWithoutClear`
- `internal/orchengine#TestWatcher_HandoffCompleteClearsThenResumes`
- `internal/orchengine#TestWatcher_HandoffSendErrorResendsSamePathOnce`
- `internal/orchengine#TestWatcher_HandoffTimeoutAborts`
- `internal/orchengine#TestWatcher_HandoffTurnEndWithoutFileNoClear`
- `internal/orchengine#TestWatcher_HardTriggerFiresOnWaitingAfterIdleGrace`
- `internal/orchengine#TestWatcher_HardTriggerKeepsIdleGraceWhenLongerThanSoftIdle`
- `internal/orchengine#TestWatcher_HardTriggerRereadBelowCapDoesNotFire`
- `internal/orchengine#TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped`
- `internal/orchengine#TestWatcher_ManualRequestBelowThresholdCyclesWithoutGrace`
- `internal/orchengine#TestWatcher_NewerEventRestartsIdleGrace`
- `internal/orchengine#TestWatcher_NoteRequestRendersTemplateFileBeforeTyping`
- `internal/orchengine#TestWatcher_PhaseForDifferentStrandIsReset`
- `internal/orchengine#TestWatcher_PreClearTurnEndNeverReadNorCyclesAgain`
- `internal/orchengine#TestWatcher_RecordsTheSessionIDItReads`
- `internal/orchengine#TestWatcher_ReloadReadsAnUnreadableStepAsThePointer`
- `internal/orchengine#TestWatcher_ReloadRestartAroundTheRetryStep`
- `internal/orchengine#TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep`
- `internal/orchengine#TestWatcher_ReloadTypesTheColorFirstAndAFailingColorNeverWedgesIt`
- `internal/orchengine#TestWatcher_RequestedCycleFiresOnSmallReadingWithoutReread`
- `internal/orchengine#TestWatcher_RequestedTriggerRecorded`
- `internal/orchengine#TestWatcher_RestartInClearingClearsOnceAndCountsOnce`
- `internal/orchengine#TestWatcher_RestartInIdleAfterResumeTimeoutStartsNoCycle`
- `internal/orchengine#TestWatcher_RestartInSoftHandoffResendsSoftStencil`
- `internal/orchengine#TestWatcher_RestartLandedInjectionNoDuplicateSend`
- `internal/orchengine#TestWatcher_RestartMidColorStepTypesItOnceWhenIdle`
- `internal/orchengine#TestWatcher_RestartUnlandedInjectionResendsOnce`
- `internal/orchengine#TestWatcher_ResumeSendErrorResendsOnce`
- `internal/orchengine#TestWatcher_ResumeTurnEndTakesFirstPostCycleReading`
- `internal/orchengine#TestWatcher_ResumeTurnNeverEndingReturnsIdleUnknown`
- `internal/orchengine#TestWatcher_SkillSkipCauses`
- `internal/orchengine#TestWatcher_SoftDeferAmongOtherTextDoesNotDefer`
- `internal/orchengine#TestWatcher_SoftDeferTurnEndAloneNeverRequalifies`
- `internal/orchengine#TestWatcher_SoftDeferWithHandoffWrittenProceedsToClearing`
- `internal/orchengine#TestWatcher_SoftDeferWithoutFileReturnsIdleAndBlocksNextAttempt`
- `internal/orchengine#TestWatcher_SoftThresholdAtOrAboveHardNeverFires`
- `internal/orchengine#TestWatcher_SoftTriggerFiresOnStopAndWaitingAfterSoftIdle`
- `internal/orchengine#TestWatcher_StaleReadingIsRereadBeforeFiring`
- `internal/orchengine#TestWatcher_StartTypesTheColorOnTheFirstIdleTickOnly`
- `internal/orchengine#TestWatcher_StrandGoneIsDone`
- `internal/orchengine#TestWatcher_StrandReboundMidTickIsNotOverwritten`
- `internal/orchengine#TestWatcher_UnknownUsageNeverTriggers`
- `internal/orchcli#orchCLI.statusCmd`
- `internal/orchengine/doc.go`
- `contracts/stencils#OrchTemplateHandoff`
- `contracts/stencils#OrchTemplateHandoffSoft`
- `contracts/stencils/orch/orch-template-handoff.md`
- `contracts/stencils/orch/orch-template-handoff-soft.md`
- `PATTERN.md`
- `pattern/PATTERN-guarded-interrupt.md`

**Delete:**
- `internal/orchengine#Session.Send`
- `internal/orchengine#Session.ClearSession`
- `internal/orchengine#Session.ReloadPlugins`
- `internal/orchengine#Session.TypeColor`
- `internal/orchengine#Session.CompactSession`
- `internal/orchengine#Session.LoadSkills`
- `internal/orchengine#deferReply`
- `internal/orchengine#idleStatePair`
- `internal/orchengine#Watcher.heldTyping`
- `internal/orchengine#Watcher.clearSendStuck`
- `internal/orchengine#paneTooShortReason`
- `internal/orchengine#paneNotIdleReason`
- `internal/orchengine#draftHoldReason`
- `internal/orchengine#heldReason`
- `internal/orchengine#Watcher.probeIdle`
- `internal/orchengine#Watcher.logHeld`
- `internal/orchengine#Watcher.logIdleStateDisagreement`
- `internal/orchengine#Watcher.typeReloadStepWhenIdle`
- `internal/orchengine#fakeSession.Send`
- `internal/orchengine#fakeSession.ClearSession`
- `internal/orchengine#fakeSession.ReloadPlugins`
- `internal/orchengine#fakeSession.TypeColor`
- `internal/orchengine#fakeSession.CompactSession`
- `internal/orchengine#fakeSession.LoadSkills`
- `internal/orchengine#phaseProbeSession`
- `internal/orchengine#phaseProbeSession.CompactSession`
- `internal/orchengine#errHeld`
- `internal/orchengine#TestCompact_HeldCompactStaysUnconfirmedAndRetypesOnTheNextTick`
- `internal/orchengine#TestCompact_TriggerTypesCompactOnlyOnIdlePass`
- `internal/orchengine#TestWatcher_AutoCompactionWaitsForIdleProbe`
- `internal/orchengine#TestWatcher_ClearErrorsUntilTimeoutResumesWithoutCounting`
- `internal/orchengine#TestWatcher_ClearErrorThenSuccessMovesOnce`
- `internal/orchengine#TestWatcher_ClearRetryWaitsForIdle`
- `internal/orchengine#TestWatcher_ClearTimeoutSendErrorResendsOnlyOncePassing`
- `internal/orchengine#TestWatcher_HandoffFileWithoutTurnEndNoClear`
- `internal/orchengine#TestWatcher_HandoffGateWaitsForIdleBeforeClear`
- `internal/orchengine#TestWatcher_HeldInjectionRecordsReasonLogsOncePerChangeWithinAHoldAndTypedLogsWait`
- `internal/orchengine#TestWatcher_HeldTypedStepRetriesOnTheNextTickWithoutAdvancing`
- `internal/orchengine#TestWatcher_IdleProbeFalseThenPasses`
- `internal/orchengine#TestWatcher_PassingProbeLeavesOtherStuckReasonsAlone`
- `internal/orchengine#TestWatcher_ReloadTypesNothingWhenIdleProbeFails`
- `internal/orchengine#TestWatcher_RestartReplayedTurnEndDoesNotOpenClearGate`
- `internal/orchengine#TestWatcher_RestartResumingInjectedTrueFailingThenPassingResendsOnce`
- `internal/orchengine#TestWatcher_ResumeSendErrorButLandedNoResend`
- `internal/orchengine#TestWatcher_ResumeSendErrorWhileTurnRunningNoResend`
- `internal/orchengine#TestWatcher_TooShortProbeRecordsStuckInIdlePhaseAndPassingProbeClearsIt`
- `internal/orchengine#TestWatcher_TurnEndBeforeHandoffFileDoesNotOpenClearGate`
- `internal/orchcli#runnerSession.Send`
- `internal/orchcli#runnerSession.ClearSession`
- `internal/orchcli#runnerSession.ReloadPlugins`
- `internal/orchcli#runnerSession.TypeColor`
- `internal/orchcli#runnerSession.LoadSkills`
- `internal/orchcli#runnerSession.CompactSession`

**Intent:**
The watcher's cycle runs over creel (decision record D4, D5, D9's hold handling).
The phase machine, its triggers (hard, requested, soft), the trigger re-read, the timeouts and the persisted phases stay as they are, but for the reload steps' timeouts and the pending step's persist time below; what changes is how each step is delivered and confirmed.
Nothing the watcher does types into the orch pane any more except the hub pause's guarded interrupt, which keeps `Session.SessionIdle` and `Session.Interrupt` (D9).

Session seam:
`Session` loses `Send`, `ClearSession`, `ReloadPlugins`, `TypeColor`, `CompactSession` and `LoadSkills`, and gains the interface method `StrandColor(guid string) (string, error)`, the strand's palette color from reed, empty for none; `SessionIdle` stays for the pause handler only.
`runnerSession` drops the six typing adapters and gains `StrandColor` over its strand status.

Triggers and hold:
A trigger fires on the existing gates minus the idle probe: a turn end is the newest event, the grace has passed, and the soft deferral hold allows it; creel's own idle check gates every delivery the cycle makes.
On entering any non-idle phase (a cycle start and the auto-compaction reload's resuming phase alike) the watcher takes a creel hold on the orch strand for `cycleHoldDuration` through `renewHold`, renews it on every tick outside idle before the pause check, so a tick held by a hub pause renews it too, and releases it through `releaseHold` at every return to idle (`toIdle`).
A restarted watcher in a non-idle phase takes the hold again before anything else.
A creel refusal of the hold starts no cycle on that tick; the watcher logs it once per change and tries again.
`orchName` resolves the orch strand's name from the GUID the state records through `Session.StrandName` at each call.

Note request and reply gate (D4):
`tickIdle` and `tickHandoff` post the note request as mail to the orch strand through `sendStep`, with the rendered handoff or soft-handoff stencil as body; `RenderHandoffInstruction` and `RenderSoftHandoffInstruction` render multi-line text, and both stencils lose their one-line constraint and name the note path, the note template and the two answers: `lyx creel post --reply <id> "written <path>"` once the note is written, or, for the soft request only, `lyx creel post --reply <id> defer`, where `<id>` is the id `lyx creel read` prints for that message (see the overview's Shared Decisions).
`State` records the request's id once the post returns.
Each tick reads the watcher's own mailbox without waiting, through `readReplies` as the read handler, which runs before the client acknowledges the read; `Run`'s start read uses the same handler.
`readReplies` counts only a reply from the orch strand's resolved name whose `reply_to` is the pending request's id; every other message, a reply to an earlier request included, is logged and dropped.
A body that is `written` followed by a path, whitespace trimmed, passes the gate when the path, resolved against the prime, is the requested note path and the file exists and is non-empty; the move to clearing or compacting is persisted inside the handler.
A reply naming another path, carrying other text, or arriving with a missing or empty file is logged and leaves the phase waiting until the handoff timeout.
`defer` declines a soft cycle when the file is still unwritten, recording `State.LastDeferral` and returning to idle with `deferred`; in a hard or requested cycle it is logged and ignored.
The gates on the `DEFER` turn-end message and on a turn end read after the file was seen go (`deferReply`, the `deferred`, `handoffWritten` and `turnEndAfterHandoff` fields of `phaseEvents`); the abort on an `EventWaiting` ask during the handoff stays, read from the events file.

Steps and outcomes:
`/clear`, `/compact <focus>`, the plugins reload, the color and the skills load are command injects (`clear`, `compact`, `reload-plugins`, `color` with `StrandColor`'s answer, `load-skills` with the space-joined list); the resume pointer and the reload pointer are text injects.
A strand with no color, `StrandColor` answering empty, queues no color inject: the reload's color step advances to the plugins step at once and the bind color counts as done, as today's "a strand with no color types nothing".
`prompt.go`'s file comment and `render`'s one-line refusal give their reason as a creel text inject being one line, no longer shuttle's `Send`.
Every step inject carries `stepInjectTTL`, ten minutes, creel's default inject TTL.
Before each post or inject `sendStep` persists the step in `State` as a `PendingStep` (whether it is mail, its command and argument or text, and the time it was persisted) with no id, then saves the id creel answers; the step's first queueing time, from which every reload step's timeout now counts (`ReloadTypedAt`), is persisted at its first queueing and kept across every requeue and restart.
A step is queued only after the tick re-read the state file and found its phase still non-idle on its strand; a re-read that finds the phase idle or bound to another strand queues nothing on that tick.
`stepOutcome` reads the pending inject's outcome from creel without blocking the tick: `injected` advances; `expired` queues the same step again and records the stall reason creel reported in `State.Stuck`; `unconfirmed` advances for `/clear`, for the color and plugins steps and for the bind color, and for a step that waits on its own signal (a turn end or a compaction boundary), which keeps waiting until its timeout; `withdrawn` queues nothing again and returns to idle, releasing the hold.
The color and plugins steps get the handoff timeout: past it the reload logs the step as skipped and moves on, as a skills step past its timeout does.
Steps that wait on the session keep reading the events file and the transcript as today (skills and retry classification, the pointer's turn end, the compaction boundary search, the resume mark), and count a turn end only when `turnEndAfterSending` finds one whose `At` second is after the second of the inject's newest `sending` record; a zero `At` never counts.
The auto-compaction entry decides as today, and its boundary is acted on at once by queueing the reload's first inject.

Restart and failed calls:
Every tick that finds a pending step with no id calls `adoptPending` before sending anything, whether a restart or a failed post or inject call of a live watcher left it so, since creel may have journaled a request whose call then failed (a non-waiting call card 4 times out, or a dropped connection, which a non-waiting call does not resume).
`adoptPending` looks the step up with `Creel.Sent` since the step's persist time and adopts only a post or inject whose command and argument, or text (a post's subject), match the step; only when none exists does it send the step.
The persist time tells the skills and retry steps apart, since the retry step is persisted only after the skills inject's outcome is read.
A restarted watcher holding an id reads its outcome and resumes without sending again.

Bind color:
`typeStartColor` queues a `color` command inject on the first tick after binding; a failed or expired one is logged and dropped, an `unconfirmed` one counts as typed, and its id lives in watcher memory only.

Other strand, failures, removals:
A failed release at a return to idle goes through `creelFailed` and does not keep the phase: the watcher returns to idle, and the hold it could not release lapses within `cycleHoldDuration`.
A phase reset because it belongs to another strand (`initCursor` through `resetForOtherStrand`) withdraws the watcher's queued injects and unread mail to the orch strand, releases its hold and drops its pending ids; a failed withdraw or release goes through `creelFailed`, saves no reset and leaves the watcher unbound, so the next tick runs the reset again; `ClearResumeNoticed` then clears, under the pause record's lock, the resume-noticed mark of each pause entry owning the strand whose resume line the withdraw removed (a withdrawn message whose subject is `pauserecord.ResumeLine`), so a continued session gets the line posted again by `queueResumeLine`.
`creelFailed` logs a failed creel call (unreachable daemon, a refusal, the sender-resolution refusal included) once per change of error and records it in `State.Stuck`; `Tick` counts it toward no consecutive tick-error cap, so a creel outage holds the cycle and never ends the watcher.
`creelFailed` also sets `State.StuckByCreel` (state key `stuck_by_creel`), which marks `State.Stuck` as a creel failure rather than a stall reason, and a restarted watcher reads it back from the state file.
A tick in which every creel call succeeded clears `State.Stuck` and `State.StuckByCreel` when the flag is set, in idle as in any other phase, so a failed release at a return to idle shows only until a tick whose creel calls all succeed, and a call that fails every tick beside one that succeeds keeps one stuck reason and logs once.
A stall reason `stepOutcome` wrote sets no `StuckByCreel`, and clears when the step advances, whether its inject reached `injected` or `unconfirmed` or the step was skipped at its timeout, and otherwise at `enter` and `toIdle` as today; a creel failure that overwrote a stall reason and then cleared leaves `State.Stuck` empty until the next `stepOutcome` writes the reason again.
A restarted watcher's first tick clears a `State.Stuck` that neither a pending step nor the `State.StuckByCreel` flag owns, so a hold reason an old watcher left in an idle state, whose `stuck_by_hold` or `stuck_by_send` flag the new `State` drops, does not linger until the next cycle's `enter`.
`State.Stuck` holds the stall reason creel last reported for the pending inject or the creel failure holding the cycle; `StuckByHold` and `StuckBySend` go from `State` (their `stuck_by_hold` and `stuck_by_send` keys leave the state file), and so do the idle probe before typing, the hold-reason bookkeeping, the `orch: injection held` logging, the session-start log moves into `recordSessionID`, and the state-versus-probe disagreement log goes (`probeIdle`, `logHeld`, `heldTyping`, `clearSendStuck`, `logIdleStateDisagreement` and their constants).
`lyx orch status`'s help (`statusCmd`) says `stuck` names the stall reason creel last reported for the watcher's pending inject or the creel failure holding the cycle.
`ResetForFreshLaunch` clears the pending step and request id.
Bound: a watcher that dies mid-cycle leaves a hold that lapses within `cycleHoldDuration`; a live cycle keeps the orch's other mail unreminded until it returns to idle, which the phase timeouts, the color and plugins steps' timeout included, bound everywhere but the clearing phase, where `lyx orch status` shows the stall.
Bound: `unconfirmed` departs from "confirms" only for `/clear`, the color and plugins steps and the bind color; every other `unconfirmed` step is still gated by its own signal.
Bound: the stamp has whole-second precision, so a step whose own turn ends within the second it was typed is missed and times out, and another turn ending in the second after the inject's newest `sending` record still counts, a window creel's idle check before `sending` keeps narrow.

PATTERN change: `PATTERN-orch-pane-single-writer`'s entry line in `PATTERN.md` becomes "Typing into the orch session from Go: only creel, through its `Delivery` seam; the orch watcher keeps only a hub pause's guarded interrupt, reed's `/rename` repair keeps its own guard, and other modules post or inject through creel.", and `pattern/PATTERN-guarded-interrupt.md`'s orch bullet says the watcher's hub-pause interrupt is the one key the watcher still sends, under that entry's exemption.
The rewrite adds 84 bytes to `PATTERN.md`, inside the room card 8's entry cuts made, so the overview ends this card under its base size and under `pattern.HeadroomBytes`; the declared text is kept verbatim.
`contracts/stencils`' doc comments on `OrchTemplateHandoff` and `OrchTemplateHandoffSoft` drop "one-line".
`internal/orchengine/doc.go` rewrites "Idle rules", the two cycle sections, "The reload sequence" and the hub pause's hold handling for creel.

Tests:
`fakeSession` loses its typing fields and methods and gains `StrandColor`; `fakeCreel` (card 9) acknowledges a mailbox read only when the handler returns nil, so a handler that fails leaves the messages unread, and `fakeCreel.Inject` can be scripted to record the inject in its sent list and still answer an error, as a daemon that journaled a request whose call then failed; the env helpers (`injectHandoff`, `reachClearing`, `reachResuming`, `finishNote`, `reachCompacting` and the rest listed) drive the cycle through `fakeCreel` from card 9: a note is finished by writing the file and answering the request with a `written <path>` reply in the fake mailbox, and a step is landed by settling its inject's outcome with a `sending` time and appending a stamped Stop to the events the fake session returns.
Every kept watcher and compact test is re-expressed over the fake creel with its behavior unchanged; the tests whose only behavior was the idle probe, a held typing or the DEFER turn-end gate go, as listed.
`TestWatcher_HandoffCompleteClearsThenResumes` and `TestWatcher_ClearCycleReloadsSkillsThenPointer` together run a clear cycle end to end over the fake creel and also assert the hold: taken at the cycle start, renewed each tick, released when the pointer's turn end ends the phase.
`TestWatcher_RestartInSoftHandoffResendsSoftStencil` restarts with the pending post absent from the fake's sent list, so the restarted watcher posts the soft stencil again; with the post present it adopts it, which `TestWatcher_RestartUnlandedInjectionResendsOnce`'s adoption rows cover.
`TestWatcher_StartTypesTheColorOnTheFirstIdleTickOnly` gains rows: a strand with no color queues no color inject; an expired or refused bind color is logged and not queued again; an `unconfirmed` one counts as typed.
`TestWatcher_ReloadTypesTheColorFirstAndAFailingColorNeverWedgesIt` gains a row: a strand with no color moves the reload to the plugins step with no color inject.
`TestWatcher_NoteRequestRendersTemplateFileBeforeTyping` asserts the posted request body names the note path, the template and both answers.
`TestWatcher_PhaseForDifferentStrandIsReset` gains rows: the reset withdraws, releases the old hold and clears the noticed mark of an entry whose resume line was withdrawn, and leaves an entry alone when the withdrawn message is not the resume line; a reset whose withdraw fails saves no reset and runs again on the next tick.
`TestWatcher_AutoCompactionReloadsPluginsThenPointer` also asserts the hold: the auto-compaction reload holds the orch through its steps.
`TestWatcher_ResumeTurnNeverEndingReturnsIdleUnknown` gains a row: an `unconfirmed` pointer keeps waiting for its turn end and returns to idle with `resume timed out` at its timeout.
`TestCompact_CompletesOnBoundaryAfterEntryAndIdle` and `TestCompact_TimeoutReturnsToIdleAndHoldsAutomaticTriggers` each gain a row for an `unconfirmed` `/compact`, which keeps searching for the boundary: the one completes on the boundary, the other times out with none.
New tests carry the creel-specific behavior:
`TestWatcher_NoteReplyGate`: a note file written with no reply posted, another path, other text, a missing or an empty file never reach `/clear`; a reply to an earlier request id or from another sender is dropped; a watcher whose handler's save fails after reading a `written` reply (the row makes the state directory unwritable from the fake's read hook) leaves the reply unread, and a new watcher over the restored directory gates on it.
`TestWatcher_StepOutcomes`: an expired step inject is queued again and its stall reason shows in `State.Stuck` until the requeued inject lands and the step advances, which leaves `State.Stuck` empty; an `unconfirmed` color or plugins inject advances the reload; an `unconfirmed` `/clear` moves to resuming; a plugins inject that keeps expiring is skipped at the handoff timeout from its first queueing, which leaves `State.Stuck` empty; a `withdrawn` outcome returns to idle and releases the hold; a watcher started over an idle state file carrying `stuck` and `stuck_by_hold` true leaves `State.Stuck` empty after its first tick.
`TestWatcher_CycleHold`: a cycle spanning a hub pause keeps renewing its hold; a refused hold starts no cycle and is tried again; a state reset to idle on another strand between the tick's load and its queueing (the fake session's events read rewrites the state file) queues nothing.
`TestWatcher_CreelFailuresHoldTheCycle`: a failing creel call holds the cycle, logs once per change, never counts toward the tick-error cap, and a sender-resolution refusal of the hold shows in `State.Stuck`; a release that fails at the return to idle still returns to idle and records the failure, and a following tick whose creel calls succeed leaves `State.Stuck` empty; an inject the fake creel journals but answers with an error leaves the pending step with no id, and the next tick adopts the journaled inject and posts or injects nothing new.
`TestWatcher_TurnEndCountsOnlyAfterSending`: an operator turn ending between a step's queueing and its newest `sending` record confirms neither the skills, the retry nor the pointer step; a turn end stamped in the second of `sending` confirms nothing; a Stop with the zero time after `sending` confirms nothing.
A Stop whose stamp line is read in one tick and whose payload in the next is pinned by card 1's `TestRunner_ReadEvents` row, since `Runner.ReadEvents` alone joins the two and this package's `fakeSession` hands over parsed events.
Restart adoption is rows of the kept restart tests:
`TestWatcher_RestartLandedInjectionNoDuplicateSend` gains a row per non-idle phase for a pending step whose id is saved: its outcome is read, nothing is posted or injected again, and the hold is taken again.
`TestWatcher_RestartUnlandedInjectionResendsOnce` gains rows for a pending step with no saved id: the step is adopted when the fake's sent list holds it and sent once when it does not.
`TestWatcher_ReloadRestartAroundTheRetryStep` and `TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep` assert that a restart at each reload step, the pointer and the retry of every skill adopts that step's own inject and never an earlier step's.
`TestCompact_*` rows cover a compact cycle through `/compact <focus>` and the boundary search, and the timeout to `compaction timed out` with no boundary; the soft cycle's `defer` reply records the deferral and a hard cycle ignores it (`TestWatcher_SoftDeferWithoutFileReturnsIdleAndBlocksNextAttempt`, `TestWatcher_DeferIgnoredInHardAndRequestedCycles`).
`TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine` keeps the interrupt behavior as today.
`TestRenderPointers_OneLineNamingTheirPaths` and `TestRenderPointers_MultiLineOverrideFails` drop their handoff rows, since the request is now mail; `TestSaveLoadState_RoundTrip` and `TestResetForFreshLaunch` cover the pending step and request id.

**ImpactSummary:** Rewrites the orch watcher's delivery and confirmation over creel inside orchengine, shrinks the Session seam (its one production adapter in orchcli changes here), and re-expresses the watcher test suite over the fake creel; no other package reads these members.
