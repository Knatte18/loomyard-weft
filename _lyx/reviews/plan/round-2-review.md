---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/02-creel-orch-receiver.md:49
    summary: Card 2 omits TestScheduler_Injects and understates the rows it changes in TestStore_InjectAndHold and TestSweep_RemovedHolders, so its own verify fails and its ImpactSummary is wrong
    fix: >-
      In _lyx/plan/02-creel-orch-receiver.md make three edits.
      (1) Add the Edit bullet `internal/creelengine_test#TestScheduler_Injects` after the `TestScheduler_Reminders` bullet.
      (2) Extend the Tests paragraph with the rows the card changes.
      `TestScheduler_Injects`' row `InjectsAreTypedBeforeRemindersInQueueOrder` drops the `«hi»` subject from its `reminderFor` entry.
      `TestStore_InjectAndHold`'s rows `Inject_SelfInjectPassesExceptForOrch` and `Inject_AndInterruptIntoOrchAreRefused` assert the refusals this card removes, so they are rewritten to the new rule (an orch strand's self-inject, and a program's inject and interrupt into an orch strand, succeed) instead of staying beside the gained rows that say the opposite.
      `TestSweep_RemovedHolders`' rows `PostWithLessThanTheBaseLeftDoesNotDelayTheSweep` and `PostInsideTheConfirmIntervalKeepsTheConfirmDeadline` read every clock wait on the premise that orch mail has no delivery worker, so they post to a non-orch strand or read only sweep waits.
      Its row `MailReturnsOneStepAtATimeThenIsUndeliveredAndAnswersItsWaiter` drops the `withheld` events from its expected list and the zero-Idle-calls assertion for `ly:orch-2`, besides the never-probed assertion the card already names.
      (3) Replace the ImpactSummary's claim that every old-reminder and `withheld` assertion lives in the four listed functions by a conclusion naming `TestScheduler_Reminders`, `TestScheduler_Injects`, `TestStore_Mail`, `TestStore_InjectAndHold`, `TestSweep_RemovedHolders`, `TestJournal_Fold` and `TestDaemon_Lifecycle`.
      The added target is admissible; checked with `lyx loom validate-plan --overlay`.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/11-orch-start-withdraws.md:47
    summary: Card 11 omits TestStart_EnsuresTheRunsStrandBeforeTheOrch, whose exact wantCalls lists break once the harness records the fake creel's calls in the same order
    fix: >-
      In _lyx/plan/11-orch-start-withdraws.md add the Edit bullet `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch` beside `TestStart_NoStrandLaunchesAndSpawnsWatcher`.
      In the Tests paragraph state that its table's `wantCalls` rows gain the fake creel's withdraw and release calls at the positions `start` makes them, since every row of that test ends in a fresh launch and so runs both rounds, and that its runs-strand ordering assertions stay.
      The added target is admissible; checked with `lyx loom validate-plan --overlay`.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-creel-orch-receiver.md:3
    summary: No card updates tools/sandbox/SANDBOX-CREEL-SUITE.md, whose scenario C9 asserts the stage 1 behavior this task removes
    fix: >-
      In _lyx/plan/02-creel-orch-receiver.md add the Edit bullet `tools/sandbox/SANDBOX-CREEL-SUITE.md` and one Intent sentence that rewrites scenario C9 (Orch).
      Its Goal, Watch and Verdict now say mail to the orch-role strand is reminded in its pane, a text inject and an interrupt from EXTERNAL are accepted, a hold from EXTERNAL or from another orch strand is refused with the program-only message, and `inbox` shows plain counts.
      The Verdict fails on a hold accepted from a non-program, or on a keystroke in an orch pane that no creel verb caused.
      The added target is admissible; checked with `lyx loom validate-plan --overlay`.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/08-drop-hub-notice-log.md:58
    summary: The shortened PATTERN-plan-generation line drops the top-level files qualifier, which the archive subdirectories under _lyx/plan make load-bearing
    fix: >-
      In _lyx/plan/08-drop-hub-notice-log.md remove the `PATTERN-plan-generation` bullet from the cut list, so its entry line keeps "`_lyx/plan/`'s top-level files hold one generation; a retired one moves to `round-<N>/prior-generation/`."
      Offset the 17 bytes with a deeper cut to `PATTERN-documentation-lifecycle`: "Design docs: none for unbuilt work; a built design lives in its package's `doc.go`." with its name and background link as they stand.
      Rewrite the card's arithmetic: the cuts total 159 bytes, `PATTERN.md` ends this card 86 bytes under its base size, and card 10's 84-byte rewrite leaves it 2 bytes under (15558 against a base of 15560).
      Update the ImpactSummary's wording to match; no `pattern/` file changes.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/01-shuttle-event-time.md:3
    summary: Card 1 edits the package doc in shuttlefake.go with no target, and creates SignalEngine with no test or consumer anywhere in the plan
    fix: >-
      In _lyx/plan/01-shuttle-event-time.md add the Edit bullets `internal/testkit/shuttlefake/shuttlefake.go` (the package doc comment has no member glyph) and `cmd/lyx/testdata/test-budget.yaml`.
      Add the Create bullet `plan:internal/testkit/shuttlefake#TestSignalEngine_PairsStampsAndHoldsTrailingStamps` -> `func TestSignalEngine_PairsStampsAndHoldsTrailingStamps(t *testing.T)`.
      Add an Intent sentence for it: over a `SignalEngine` whose `ParseEventsFn` reads one line, a stamped Stop line yields a turn-end signal with `Event` set and the stamp's time, a Stop with no stamp or a malformed one yields the zero time, `consumed` stops before a trailing run of stamp lines, and a plain `Engine` yields no signals.
      Raise the `internal/testkit/shuttlefake` row of `cmd/lyx/testdata/test-budget.yaml` by one.
      Admissible together; checked with `lyx loom validate-plan --overlay`.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/10-orch-cycle-over-creel.md:312
    summary: The new TestWatcher_RestartAdoptsThePendingStep repeats restart-per-step behavior that five kept restart tests already cover
    fix: >-
      In _lyx/plan/10-orch-cycle-over-creel.md remove the Create bullet for `plan:internal/orchengine#TestWatcher_RestartAdoptsThePendingStep` and move its coverage into rows of the restart tests the card already re-expresses.
      `TestWatcher_RestartLandedInjectionNoDuplicateSend` gains the rows for a pending step whose id is saved, in each non-idle phase: its outcome is read, nothing is posted or injected again, and the hold is taken again.
      `TestWatcher_RestartUnlandedInjectionResendsOnce` gains the rows for an unsaved id: the step is adopted when the fake's sent list holds it and sent once when it does not.
      `TestWatcher_ReloadRestartAroundTheRetryStep` and `TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep` assert that a restart at each reload step, the pointer and the retry of every skill adopts that step's own inject and never an earlier step's.
      Rewrite the card's two sentences that name the removed test (the soft-handoff restart sentence and the restart paragraph) to name these rows.
      The removal is admissible; checked with `lyx loom validate-plan --overlay`.
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/plan/10-orch-cycle-over-creel.md:268
    summary: The stall reason an expired step writes to State.Stuck has no stated clearing rule, unlike the creel-failure reason
    fix: >-
      In _lyx/plan/10-orch-cycle-over-creel.md add one sentence after the paragraph that states the creel-failure clearing rule.
      A stall reason `stepOutcome` wrote clears when that step's inject reaches `injected` or `unconfirmed` and the step advances, and otherwise at `enter` and `toIdle` as today; a stall reason sets no `StuckByCreel`.
      A creel failure that overwrote a stall reason and then cleared leaves `State.Stuck` empty until the next `stepOutcome` writes the reason again, and `TestWatcher_StepOutcomes` asserts the clearing at the step's advance.
---

### [BLOCKING] Card 2 names too few of the tests its change breaks

**Location:** `_lyx/plan/02-creel-orch-receiver.md:49` (Tests paragraph) and line 58 (ImpactSummary).
**Issue:**
Card 2 removes the orch refusal, the `withheld` line and the reminder subject, and its ImpactSummary concludes that every assertion on the old reminder text and the `withheld` record lives in the four creelengine test functions listed.
`internal/creelengine/deliver_test.go` shows that is false: `TestScheduler_Injects` asserts `reminderFor(1, "#1 from "+alpha+" «hi»")` in its first row, and it is not in the card's Edit list.
`internal/creelengine/verbs_test.go` holds two `TestStore_InjectAndHold` rows, `Inject_SelfInjectPassesExceptForOrch` and `Inject_AndInterruptIntoOrchAreRefused`, that assert the refusals the card removes; the card says only that the function "gains rows", so old and new rows would contradict each other.
Three `TestSweep_RemovedHolders` rows rest on the removed behavior: two read unfiltered clock waits under the comment that orch mail "has no delivery worker", which the card's `deliverableReceiver` change falsifies, and one expects `withheld` events and zero `Idle` calls for `ly:orch-2`.
The card's verify (`./internal/creelengine`) therefore fails as written, and the card's test list gives the implementer no pointer to the rows.

### [BLOCKING] Card 11 omits a start test whose exact call list breaks

**Location:** `_lyx/plan/11-orch-start-withdraws.md:47` (Tests paragraph).
**Issue:**
Card 11 has the harness's `fakeCreel` record its calls "in the harness order beside adds, removes and starts".
`TestStart_EnsuresTheRunsStrandBeforeTheOrch` in `internal/orchcli/start_test.go` compares `h.calls` to exact lists such as `{"add","start"}`, and every row of it ends in a fresh launch that now runs both withdraw rounds.
The card lists the other `TestStart_*` tests it changes but not this one, so the card's verify fails and the Edit list gives no authority for the rows to change.

### [MEDIUM] The creel sandbox suite still specifies stage 1 behavior for orch strands

**Location:** `tools/sandbox/SANDBOX-CREEL-SUITE.md:243` (scenario C9) against `_lyx/plan/02-creel-orch-receiver.md:3`.
**Issue:**
Scenario C9 sends mail, an inject and an interrupt to an orch-role strand and expects the mail stored with a `withheld` record, the inject and interrupt refused, `inbox` showing the mail as withheld and a `FAIL` verdict on any keystroke in an orch pane.
Decision record D1 reverses all four, and the sandbox suite is the black-box check of creel's behavior, so a run after this task fails on correct behavior.
No card targets the file; card 2 is the card that lifts the refusal.
This breaks no gate, which is why it is MEDIUM.

### [LOW] The plan-generation cut drops a qualifier the archive directories make load-bearing

**Location:** `_lyx/plan/08-drop-hub-notice-log.md:58`.
**Issue:**
Card 8 shortens `PATTERN-plan-generation` to "`_lyx/plan/` holds one generation".
`pattern/PATTERN-plan-generation.md` says only the directory's top-level files hold one generation, because Plan-Write's `archive-*/` rotation keeps superseded plans in subdirectories of `_lyx/plan/`; the shortened line reads as if the directory holds no archive.
The card's ImpactSummary leaves this meaning check to review.
The other seven cuts keep their meaning, and the headroom arithmetic holds as written (cuts of 167 bytes against additions of 157 leave `PATTERN.md` at 15550, under its 15560-byte base); the fix keeps the qualifier and finds the room in the documentation-lifecycle line.

### [LOW] Card 1's kit change has an untargeted doc edit and an untested type

**Location:** `_lyx/plan/01-shuttle-event-time.md:3` and line 30.
**Issue:**
The Intent has the package doc in `shuttlefake.go` name `SignalEngine`, but no target covers that file; the doc comment has no member glyph, so the file self glyph is the admissible spelling (`internal/shuttleengine/doc.go` is listed the same way).
`SignalEngine` carries the stamp pairing and the hold-back of trailing stamps, yet no card builds one: card 10's tests use the in-package `fakeSession`, and the kit's other types have tests in `shuttlefake_test.go`.
Untested kit code is where a second, silent copy of the stamp rule would drift from the production reading.
The kit's inert-by-default contract and `TestEngine_InertDefaults` are honored, and every other kit consumer reads what it reads today because `Engine` gains no parser.

### [LOW] A new restart test repeats five kept ones

**Location:** `_lyx/plan/10-orch-cycle-over-creel.md:312`.
**Issue:**
`TestWatcher_RestartAdoptsThePendingStep` asserts that a watcher restarted in every non-idle phase, with the pending id saved or not, does not post or inject twice, and that each reload step adopts its own inject.
`TestWatcher_RestartLandedInjectionNoDuplicateSend`, `TestWatcher_RestartUnlandedInjectionResendsOnce`, `TestWatcher_RestartInClearingClearsOnceAndCountsOnce`, `TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep` and `TestWatcher_ReloadRestartAroundTheRetryStep` already pin restart-per-phase and restart-per-reload-step behavior, and card 10 keeps and re-expresses all five.
The new mechanism, adoption through the journal, is an input to those same observable outcomes, so `PATTERN-test-economy` makes it rows of the existing tests.

### [LOW] The expired-step stall reason has no clearing rule

**Location:** `_lyx/plan/10-orch-cycle-over-creel.md:268` and lines 286 to 288.
**Issue:**
`State.Stuck` now carries two kinds of text: a creel failure, flagged by `StuckByCreel` and cleared by the first tick whose creel calls all succeed, and the stall reason `stepOutcome` records for an expired inject.
The second has no stated end beyond the phase change that `enter` and `toIdle` already perform, so a reason written during the color step can stay in `lyx orch status` through the plugins, skills and pointer steps after the step advanced.
The creel-failure reason is covered as the focus directive asked; this is the other occupant of the field.

## Rejected

- Focus 1, PATTERN.md room: the cards that rewrite entry lines are 8 and 10, and card 8 cuts the room.
  Measured against `PATTERN.md` at the merge base (15560 bytes, `pattern.HeadroomBytes` 15564): card 8's eight cuts save 167 bytes, its durable-state rewrite adds 73 and card 10's single-writer rewrite adds 84, so the file ends at 15550, 10 under the base, and stands at 15466 after card 8 alone.
  Both declared texts are kept verbatim and every cut line stays under the 320-character entry cap.
  Not a defect; F4 concerns only one cut's meaning.
- Focus 3, card 10's resend after a journaled failure: `adoptPending` runs on every tick that finds a pending step with no id before anything is sent, looks the step up by command and argument, text or subject since the persist time, and the journal keeps nanosecond times and the subject as the body's first line, so the match holds for the note request, the injects and the retry step.
  A lookup that itself fails goes through `creelFailed` and sends nothing.
  The creel-failure stuck reason has its clearing rule (`StuckByCreel`, cleared by a tick whose creel calls all succeed).
  Not a defect; F7 concerns the other reason.
- Focus 4, bounds and ways forward: cards 2, 3, 4, 5, 7, 9, 10 and 11 state what each change can skip and what bounds it, and card 2's hold refusal, card 6's returned error and card 11's first-withdraw failure each name a step the caller can take from the pane that reaches it.
  The refusals in cards 3 and 4 for a non-program sender name the rule only; they are reached by Go code lyx ships or a hand-built request, and no operator pane follows, so they are not flagged.
- Card 10's size: the `Session` seam change breaks every watcher test at once, so the card cannot split along the compile boundary; not raised.
- `docs/overview.md` edits in cards 6 and 7 are guarded by the link test in `internal/lyxcwd`, which verify does not run; `docs/` is not a package and the cards add no link, so not raised.
