# Round 3 fixer-report — plan

## Changes

- F1 (MEDIUM), fixed as proposed: card 11's Watcher paragraph says the retry rewrites `State.Stuck` on each failing tick without setting `State.StuckByCreel`, and the tick in which both retries succeed clears `State.FreshLaunchFailure` and `State.Stuck` in the same state save.
  The `TestWatcher_CreelFailuresHoldTheCycle` retry row asserts the failure in `State.Stuck` with `StuckByCreel` unset while failing, and `State.Stuck` empty after the successful retry.
- F2 (MEDIUM), fixed as proposed: card 9's `injectOutcomeOf` returns the time of the newest `sending` record for the id, with the reason (`markSending` per attempt, `requeue` after `ErrBusy` in memory only); the `CreelOutcome` definition says the same.
  `TestCreelSeam_FoldsTheJournal` asserts the later time for an id with two `sending` records.
  Card 10's `turnEndAfterSending` sentence, its Bound line and the `TestWatcher_TurnEndCountsOnlyAfterSending` row say "newest `sending` record".
- F3 (LOW), fixed as proposed: card 10's clearing paragraph adds that a restarted watcher's first tick clears a `State.Stuck` that neither a pending step nor the `StuckByCreel` flag owns; card 11 adds `State.FreshLaunchFailure` as an owner to that rule.
  `TestWatcher_StepOutcomes` gains a row: a watcher started over an idle state file with `stuck` and `stuck_by_hold` true leaves `State.Stuck` empty after its first tick.
- F4 (LOW), fixed as proposed: card 10 moves the note-request body assertion into `TestWatcher_NoteRequestRendersTemplateFileBeforeTyping` and drops the `written` reply row; moves the phase-reset rows into `TestWatcher_PhaseForDifferentStrandIsReset`, the auto-compaction hold into `TestWatcher_AutoCompactionReloadsPluginsThenPointer`, the unconfirmed-pointer row into `TestWatcher_ResumeTurnNeverEndingReturnsIdleUnknown`, and the unconfirmed-compact row into both `TestCompact_*` tests named.
  No target changes.
- F5 (LOW), fixed as proposed: card 3's `TestScheduler_Injects` gains a row asserting a text, command, text inject sequence is typed in queue order, read from the journal's `sending` events.

## Self-check

- fix-consistency, MEDIUM: card 10's restart rule named `State.FreshLaunchFailure`, a field card 11 adds — fixed as proposed: card 10 names only pending step and `StuckByCreel`; card 11 adds the field as an owner of that rule.
- fix-consistency, LOW: card 11's "card 10's clearing rules cover only ..." omitted the new restart rule — fixed as proposed.
- fix-consistency, LOW: `TestWatcher_TurnEndCountsOnlyAfterSending` row kept "its `sending` record" — fixed as proposed.
- fix-affected-set, LOW: same `TestWatcher_TurnEndCountsOnlyAfterSending` row — duplicate of the fix-consistency finding, fixed there.
- fix-claims: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
