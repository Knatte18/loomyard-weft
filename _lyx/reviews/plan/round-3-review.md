---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-orch-start-withdraws.md:44
    summary: the State.Stuck text the fresh-launch retry writes has no clearing rule once the retry succeeds, so it outlives the failure that wrote it
    fix: >-
      In card 11's Watcher paragraph (the one beginning "Watcher (`Watcher.tick` through `retryFreshLaunch`)"),
      state that the tick in which both retries succeed clears `State.FreshLaunchFailure` and `State.Stuck` in the same state save,
      and that while the field is set the retry rewrites `State.Stuck` on each failing tick without setting `State.StuckByCreel`.
      Extend the `TestWatcher_CreelFailuresHoldTheCycle` row that "then clears the field" to assert `State.Stuck` is empty after the successful retry.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/09-orch-creel-seam.md:119
    summary: injectOutcomeOf does not say which sending record's time it returns, and creel journals a new sending record on every ErrBusy requeue, so the first one would let a turn end before the final typing confirm a step
    fix: >-
      In card 9, where `injectOutcomeOf` is described (the `log --id` fold), say the `CreelOutcome` time is that of the newest `sending` record for the id,
      because `markSending` journals one `sending` line per typing attempt and `requeue` after `ErrBusy` only resets in-memory state.
      Add a row to `TestCreelSeam_FoldsTheJournal` with two `sending` records for one id (a busy attempt, then the typed one) and assert the later time.
      In card 10's paragraph on `turnEndAfterSending` (line 269) and the matching Bound line, change "the inject's `sending` record" to "the inject's newest `sending` record".
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/10-orch-cycle-over-creel.md:286
    summary: a state file written by the old binary with stuck_by_hold or stuck_by_send set keeps its stuck text with no flag after the upgrade, and nothing clears it in the idle phase
    fix: >-
      In card 10's clearing paragraph (lines 286-287), add that a restarted watcher's first tick clears a `State.Stuck` that no pending step, no `StuckByCreel` flag and no `FreshLaunchFailure` owns,
      so a hold reason an old watcher left in an idle state does not linger until the next cycle's `enter`.
      Add one row to `TestSaveLoadState_RoundTrip` or `TestWatcher_StepOutcomes` that loads a state file carrying `stuck` and `stuck_by_hold` true and asserts `State.Stuck` is empty after the first tick.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/10-orch-cycle-over-creel.md:308
    summary: three new test functions carry rows that an existing kept test already pins, against PATTERN-test-economy
    fix: >-
      Move these rows into the named existing tests (already in card 10's Edit list) and keep only the genuinely new rows in the new functions.
      `TestWatcher_NoteReplyGate`: drop the "posted body names the note path, the template" assertion into `TestWatcher_NoteRequestRendersTemplateFileBeforeTyping` (add the two answers there) and drop the "a `written <path>` reply passes and `/clear` is queued" row, which `TestWatcher_HandoffCompleteClearsThenResumes` already runs through the helpers' reply.
      `TestWatcher_CycleHold`: move the phase-reset rows (withdraw, release, noticed-mark clearing, failed reset runs again) into `TestWatcher_PhaseForDifferentStrandIsReset`, and the auto-compaction hold assertion into `TestWatcher_AutoCompactionReloadsPluginsThenPointer`, as the plan already does for the clear cycle at line 303.
      `TestWatcher_StepOutcomes`: move the unconfirmed-pointer row into `TestWatcher_ResumeTurnNeverEndingReturnsIdleUnknown` and the unconfirmed-compact row into `TestCompact_TimeoutReturnsToIdleAndHoldsAutomaticTriggers` and `TestCompact_CompletesOnBoundaryAfterEntryAndIdle`.
      No target changes, so the plan gate's target checks are not affected.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/03-creel-command-inject.md:80
    summary: the scheduler rows never assert that a command inject is typed in queue order with text injects, and the fake records commands apart from sends so a test cannot read the interleaving from it
    fix: >-
      In card 3's `TestScheduler_Injects` rows, add one row that queues a text inject, a command inject and a text inject to a live orch strand and asserts their order
      from the journal (the `sending` events of the three ids appear in queue order), since `creelfake.Delivery` keeps `Sends()` and `Commands()` as separate lists.
      This covers the decision record's "ordered with other injects" at the typing level and needs no change to the fake.
---

### [MEDIUM] Fresh-launch retry failure text has no clearing rule

**Location:** `_lyx/plan/11-orch-start-withdraws.md:44`
**Issue:** The watcher paragraph says the failure shows in `State.Stuck`, "which no other successful creel call clears while the field is set", and that card 10's clearing rules "cover only `creelFailed`'s reasons and `stepOutcome`'s stall reasons".
So this reason is under neither rule.
Once both retries succeed the field is cleared, but nothing clears the text: the relaunched watcher sits in idle, where `enter` and `toIdle` do not run until the next cycle, and the all-success tick only clears a `State.StuckByCreel` reason.
`lyx orch status` keeps reporting a fresh-launch failure that no longer exists.
The test row for the retry ("then clears the field") does not assert `State.Stuck`, so nothing would catch it.

### [MEDIUM] Which sending record's time `injectOutcomeOf` returns is unstated

**Location:** `_lyx/plan/09-orch-creel-seam.md:119`
**Issue:** In `internal/creelengine/deliver.go`, `worker.typeInject` calls `markSending` before every typing attempt, and on `ErrBusy` `requeue` resets only the in-memory outcome.
An inject that meets a held or busy session therefore has several `sending` lines in the journal before the one that is typed.
Card 9 says `injectOutcomeOf` reads "the `sending` record's time" and card 10 gates the skills, retry and pointer steps on a turn end stamped after that second.
Taking the first `sending` record lets an operator turn that ends between a busy attempt and the final typing confirm the step, which is the false confirmation the stamp rule exists to prevent; the Bound at card 10 line 293 calls the window narrow, but with requeues it spans the whole busy period.

### [LOW] A stale idle-probe stuck reason survives the upgrade

**Location:** `_lyx/plan/10-orch-cycle-over-creel.md:286`
**Issue:** `State` loses `StuckByHold` and `StuckBySend`, and the old watcher cleared a flagged reason on restart or on a passing probe.
An idle-phase state file written by the old binary (a too-short probe or a held bind color) loads with `stuck` set and no flag.
The new clearing rules cover only `StuckByCreel` reasons, step stall reasons cleared at `enter`/`toIdle`, and the fresh-launch field, so that text stays until the next cycle starts.
The binary-change restart happens at an idle, restart-safe tick, which is where such a state is most likely to be.

### [LOW] New watcher tests duplicate rows of kept tests

**Location:** `_lyx/plan/10-orch-cycle-over-creel.md:308`
**Issue:** `PATTERN-test-economy` says a new test function must cover a behavior no existing test covers, else it becomes rows.
Existing kept tests already pin: the request naming the template and note path (`TestWatcher_NoteRequestRendersTemplateFileBeforeTyping`), the reply-driven clear (`TestWatcher_HandoffCompleteClearsThenResumes`, whose helpers now answer with a `written` reply), the reset for another strand (`TestWatcher_PhaseForDifferentStrandIsReset`), the auto-compaction reload (`TestWatcher_AutoCompactionReloadsPluginsThenPointer`), the pointer that never ends (`TestWatcher_ResumeTurnNeverEndingReturnsIdleUnknown`) and the compact timeout and boundary (`TestCompact_TimeoutReturnsToIdleAndHoldsAutomaticTriggers`, `TestCompact_CompletesOnBoundaryAfterEntryAndIdle`).
The plan applies the row-extension approach to the clear cycle's hold (line 303) but not to the auto-compaction reload's.
The genuinely new rows (other path, other text, missing or empty file, earlier request id, other sender, failed save, creel failures, stamp rules, adoption) stay in the new functions.

### [LOW] Command-inject typing order is not asserted

**Location:** `_lyx/plan/03-creel-command-inject.md:80`
**Issue:** Decision D3 says a command inject is "ordered with other injects".
Card 3 asserts queue order only at the store level (`Queued`), and its scheduler rows type one command at a time.
`creelfake.Delivery` records `Sends()` and `Commands()` separately, so a test reading the fake cannot tell whether a command was typed between two text injects.
The journal gives the order without any change to the fake.

## Rejected

- Unnamed tests asserting removed behavior: every `Test` function in `internal/orchengine` that card 9, 10 or 11 does not name was checked (the notice-queue tests go with `notice_test.go`; the rest use neither the removed `Session` methods nor `NewWatcher`), and every creelengine, creelcli, battenshed, battencli, loomcli, tenderengine, shedrun and envkit test that asserts the orch refusal, the `withheld` record, the reminder subject, the notice log or the old revive fields is named by its card.
- Fakes recording new calls: `creelfake.Delivery.Command` records into its own list, so no existing `Sends()` comparison changes; the only test comparing `startHarness.calls` exactly is `TestStart_EnsuresTheRunsStrandBeforeTheOrch`, which card 11 names; `flakySession` embeds `fakeSession` and needs no edit.
- PATTERN.md size: the seven entry cuts remove 159 bytes and the two rewrites add 73 and 84, so the file ends at 15558 against a merge-base size of 15560 and a headroom line of 15564, and sits 86 under the base after card 8; every rewritten entry line is under the 320-character cap.
- Lost qualifiers in the seven shortened PATTERN lines: each dropped detail is stated in its background file (envelope keys, the real process, the negative answers); `PATTERN-shed-recipe-registry` has no background file but its shortening keeps its meaning.
- Test budget: each package's top-level test count equals its budget row today, and the cards that add a top-level test (shuttlefake, tendercli, orchcli twice) raise the row; orchengine ends below its row.
- Reachability of prescribed refusals, attack-surface statements, card order and verify coverage showed no defect.
