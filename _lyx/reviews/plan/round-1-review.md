---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-start-send-teardown.md
    summary: Card 5 leaves awaitStartup's three consecutive-status-failure returns (wait.go errStrandNotTracked, errStrandPaneBindingCleared, reed status failed) returning run.identity() with the record reading running, though D4 says every start failure past AddStrand takes the teardown
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-webster-stops-through-shuttle.md
    summary: Card 9 says the Reed fields of RunDeps, BeginDeps and RecoverDeps stay for the remaining StrandLive reads, but only RecoverDeps reads StrandLive; RunDeps.Reed and BeginDeps.Reed are left as dead fields no card deletes
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/09-webster-stops-through-shuttle.md
    summary: Cards 8 and 9 route existing strand removals onto the stop verb without stating in Intent what that lets through; in card 9 awaitTerminal treats a stop failure as a warning, so a record-write failure now leaves a live recovery strand behind a terminal digest where the old removal did not
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/06-gate-boundary-idle.md
    summary: Card 6 keys boundaryIdle on session signals alone, while Shared Decision idle-reading-capability says an engine without SessionCycler reads no session signals because the 10 s override is timed by the idle reading
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/04-send-settles-and-confirms.md
    summary: Card 4's clear rows need the confirm test's engine to carry SessionCycler as well as InputBoxClearer, which then also runs card 3's idle wait before typing; the card names only a scripted InputBoxClearer on that engine
  - id: F6
    severity: LOW
    class: design
    location: _lyx/plan/03-send-waits-for-idle.md
    summary: Card 3's idle-wait table row for an engine without the idle reading keeping today's refusal duplicates TestRun_InterruptAndSend_RefuseAgentlessShellPane, which already pins Send wrapping ErrPaneNotReady on such an engine
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/08-burler-stops-through-handle.md
    summary: Card 8's Intent and ImpactSummary count three test files and six callers of burlerengine.New, but four test files build an engine, so New has seven callers; the target list itself is complete
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/05-start-send-teardown.md
    summary: Card 5 re-purposes abandonStartup for send and persist causes but does not widen its doc comment and Warn, which say the provider never became ready, and counts three existing calls passing nil while loadSkillsThenPrompt holds two more
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/plan/03-send-waits-for-idle.md
    summary: Card 3 updates resumeParkedDriver's doc comment but not driverResumeSendAttempts' comment, which still says the attempts bound the wait for an input-ready pane, now true only for an engine without the idle reading
---

### [MEDIUM] Card 5 misses awaitStartup's status-failure exits

**Location:** `_lyx/plan/05-start-send-teardown.md`; code at `internal/shuttleengine/wait.go:484-496`.

**Issue:** D4 says every start failure past `AddStrand` takes the `abandonStartup` teardown, "so no start error leaves a live strand behind a record reading `running`".
Card 5 routes `loadSkillsThenPrompt`'s failures and lists `Run.awaitStartup` as a target, but only re-signs its three existing `abandonStartup` calls.
`awaitStartup`'s three consecutive-status-failure returns (`errStrandNotTracked`, `errStrandPaneBindingCleared`, and the default "reed status failed" case) return `run.identity(), err` without finalizing, so the record stays `running`.
In the default case reed could not report the strand, so it may well be live: the #444 shape, a young `running` record over a strand attach cannot account for.
These returns already check `allOutputFilesExist` first, so routing them would not break the completion signal.

**Fix:** Either route these three returns through `abandonStartup(OutcomeDied, err)`, with a row in card 5's tests, or state in card 5's Intent why each is outside D4 (for example, no live strand is possible when reed reports it untracked) and what the attach probe then does with the `running` record.

### [MEDIUM] Card 9 leaves RunDeps.Reed and BeginDeps.Reed dead

**Location:** `_lyx/plan/09-webster-stops-through-shuttle.md`; code at `internal/websterengine/runlevel.go:130,413`, `beginbatch.go:91,395`, `recoverbatch.go:77,550`.

**Issue:** The card says the three deps structs' `Reed` fields "stay for the liveness reads that remain (`StrandLive`)".
In production code `RunDeps.Reed` is read only at `runlevel.go:413` (the `reclaimEntryTimeStrands` call card 9 moves to the stopper) and `BeginDeps.Reed` only at `beginbatch.go:395` (the prior-recovery reclaim card 9 moves).
Only `RecoverDeps.Reed` keeps a reader, `StrandLive` at `recoverbatch.go:550`.
After card 9 two exported fields are set by `webstercli`'s `run.go` and `beginbatch.go` and by test fixtures, but read by nothing, and card 10 does not delete them.

**Fix:** Have card 9 (or card 10) delete `RunDeps.Reed` and `BeginDeps.Reed` with their fills in `webstercli/run.go`, `webstercli/beginbatch.go`, `loomcli`'s `wire` and the `runlevel_test.go`/`beginbatch_test.go` fixtures, and correct the sentence to say only `RecoverDeps.Reed` stays.

### [MEDIUM] Cards 8 and 9 state no bound for the rerouted stops

**Location:** `_lyx/plan/09-webster-stops-through-shuttle.md`, `_lyx/plan/08-burler-stops-through-handle.md`; code at `internal/websterengine/recoverbatch.go:580-586`.

**Issue:** Each card moves an existing strand removal onto a verb that now writes a record first and, per card 7, removes nothing when that write fails.
Neither card's Intent states what that lets through at its own call sites; both lean on card 7's bound.
For card 9 the difference is observable: `awaitTerminal` treats a removal failure as a warning and carries on through the terminal digest.
With `StopStrand`, a `run.json` write failure now leaves the recovery strand live while `awaitTerminal` proceeds, where today's raw removal would have ended it.
Under D7 the crash case (a terminal record over a live strand) is bounded by attach, but this is not a crash: the record stays `running` and the strand stays live.
For card 8 the change is milder: `stopHalf` and `stopLiveHalf` already fail with their way forward, but a reader of the card cannot tell that a record-write failure now leaves the half's strand running.

**Fix:** State in each card's Intent what the rerouted stop can now leave behind and what bounds it.
For `awaitTerminal`, decide whether a stop-verb error stays a warning or becomes a refusal now that the strand may still be live, and say which.

### [LOW] Card 6's boundary check disagrees with the idle-reading Shared Decision

**Location:** `_lyx/plan/06-gate-boundary-idle.md`, `_lyx/plan/00-overview.md` (Shared Decision `idle-reading-capability`).

**Issue:** The Shared Decision says an engine without `SessionCycler` "reads no session signals, since D1's 10 s override is timed by the idle reading".
Card 6's `boundaryIdle` acts "for an engine with session signals", and says "an engine without session signals answers idle at once".
An engine with `SessionSignalParser` but no `SessionCycler` would then clear a boundary on an unmatched `turn_start` and never get the 10 s release, holding the gate until a turn end, a `TurnStartInterrupt`, or the run deadline.

**Fix:** Key `boundaryIdle` on the same capability pair as card 3's `awaitIdleSession`: idle reading plus session signals, with an engine lacking `SessionCycler` answering idle at once.

### [LOW] Card 4's clear rows need the idle reading on the confirm engine

**Location:** `_lyx/plan/04-send-settles-and-confirms.md`; code at `internal/shuttleengine/run_test.go:914-937`.

**Issue:** The clear runs only for an engine with both `SessionCycler` and `InputBoxClearer`.
Card 4 rewrites `TestRun_Send_ConfirmsSubmission` "with a scripted `InputBoxClearer` on its engine", meaning `inputBoxEngine`, which wraps `fakeEngine` and has no `SessionCycler`.
Card 3's idle-reading fake is a separate engine in `fakes_test.go`.
As written, the clear rows (doubled replay, needle-long prefix, placeholder each cleared to empty) cannot reach the clear.
Giving `inputBoxEngine` the idle reading also makes card 3's idle wait run before typing in every row, which reads the pane and must be scripted.

**Fix:** State that the confirm test's engine gains the idle reading (or composes card 3's fake) for the clear rows, keeping a row without it for "an engine without the idle reading getting no clear".

### [LOW] Card 3 plans a row an existing test already pins

**Location:** `_lyx/plan/03-send-waits-for-idle.md`; code at `internal/shuttleengine/run_test.go:702-724`.

**Issue:** Card 3's new idle-wait table includes "an engine without the idle reading keeping today's refusal".
`TestRun_InterruptAndSend_RefuseAgentlessShellPane` already drives `Run.Send` on a `fakeEngine` with no `SessionCycler` against a not-ready pane and asserts the refusal wraps `ErrPaneNotReady`.
That test keeps passing unchanged under card 3, so the new row covers no new behaviour (`PATTERN-test-economy`).

**Fix:** Drop the row and name `TestRun_InterruptAndSend_RefuseAgentlessShellPane` as the test that pins the unchanged guard.

### [LOW] Card 8 miscounts the callers of burlerengine.New

**Location:** `_lyx/plan/08-burler-stops-through-handle.md`.

**Issue:** The Intent says "the three test files that build an engine" and the ImpactSummary says "all six callers (three production wirings, three test files)".
Four test files call `New`: `burlerengine/engine_test.go:393`, `burlercli/wiring_test.go:666`, `burlerengine/smoke_round_test.go:322` and `burlerengine/smoke_cluster_test.go:140`.
All four are in the target list, so only the stated count is wrong.

**Fix:** Say four test files and seven callers, or drop the count and name the files.

### [LOW] Card 5 does not widen abandonStartup's own wording

**Location:** `_lyx/plan/05-start-send-teardown.md`; code at `internal/shuttleengine/wait.go:523-565`.

**Issue:** `abandonStartup`'s doc comment calls it "awaitStartup's not-ready teardown: the provider never reached StartupReady", and its Warn reads "provider never became ready; strand torn down".
Card 5 routes send-busy, send-not-landed and persist failures through it, so both become false for those causes, and the card says nothing about either.
The card also says "its three existing calls pass nil", but besides awaitStartup's three there are two more in `loadSkillsThenPrompt` (`run.go:501,515`, the `skillLoader` error and the died load turn), and the card does not say which cause each passes.

**Fix:** Have card 5 widen the doc comment and the Warn to cover a start that never took its first input, and name the cause each of `loadSkillsThenPrompt`'s two existing calls passes.

### [NIT] driverResumeSendAttempts' comment goes stale

**Location:** `_lyx/plan/03-send-waits-for-idle.md`; code at `internal/loomcli/driverresume.go:17-22`.

**Issue:** Card 3 updates `resumeParkedDriver`'s doc comment because, against Claude, a not-ready pane now waits inside the send and fails `ErrSessionBusy`, which the loop does not retry.
The constants' comment still says they "bound the wait for a parked driver's pane to be input-ready: about a minute in all", which now holds only for an engine without the idle reading.

**Fix:** Add the constants' comment to card 3's loomcli doc update.

## Verdicts on the focus directives

- Card 8 granularity: one reviewable unit.
  The engine's `stopHalf` and the adapter's `stopLiveHalf` share one seam, wired from one `burlerRemover` local in `loomcli`'s `wire`, so splitting the `shedadapters`/`shedrecipe` removal off would leave an intermediate card wiring a seam only one side uses.
  Every caller of the dropped remover is reached: `handoff.go`'s `stopHalf` is the `Engine.stopHalf` target, `shedadapters/burler.go:588` is `stopLiveHalf`, and every test file using `shedfake.StrandRemover` or `NewReedStrandRemover` is listed.
- `InputBoxReader` implementers: `Claude` and `run_test.go`'s `inputBoxEngine` are the only ones (no `InputBoxText` method exists in any other fake), so card 2's list is complete; card 4's fake gap is F5.
- Constraints: `PATTERN-shuttle-provider-seam` is carried by cards 2 and 4, `PATTERN-transient-stop` by card 5, `PATTERN-completion-signal` by card 7, `PATTERN-spawn-observability` by card 7's Warn, and `PATTERN-test-speed`/`PATTERN-test-economy` by the cards' `fakeClock` tests.
  `PATTERN-wait-mark-display-only` and `PATTERN-orch-pane-single-writer` are negative constraints that no card touches, which satisfies them; `PATTERN-comment-line-breaks` binds every card through the gate's lint.
  Card 10's grep-and-reword step is part of deleting the removers D7 retires, and card 7's `saveState` refactor of `notifyHeld` and `checkLivenessTick` is exactly what the post-Discussion "D7 record write path" decision licenses; neither adds scope.
- D8: card 9's reason for adding no scan test (the remaining raw `RemoveStrand` callers would make the test a list of exceptions) is the judgement the record leaves to the plan.
  `PATTERN-shuttle-stop` lands in card 9, after attach (card 7), burler (card 8) and webster (card 9) have all moved, so it is true when it lands.
- Card order: cards 5 and 6 use only symbols cards 3 and 4 create, card 3 only creates the `sendContext.deadline` field that card 6 sets, and card 7's marked-handle `finalize` is never reached from `abandonStartup`, since no handle exists during start; no card depends on a later one.
- Test economy, card 4 and card 7: card 4 extends `TestRun_Send_ConfirmsSubmission` and `TestRun_Send_DeliveryVerification` instead of adding functions.
  `stop_test.go` pins a new surface; its attach-facing cases go into the existing `TestAttach_OutcomeDisposition` and `TestAttach_RemovesSupersededStrands`; no duplication found beyond F6.
- Verify: `go test ./...` and `go test -tags integration ./...` cover every targeted package, `tmux` and `llm` are compiled by `go vet`, and the order matches the plan template.
