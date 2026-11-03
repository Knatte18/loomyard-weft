# Fixer-report, round 1 (code)

Every finding of `round-1-review.md` is fixed; the gate (`lym gate test ./internal/shuttleengine ./internal/shuttleengine/claudeengine`) passes with no failures.

## Changes

- **F1 (BLOCKING), `pattern/PATTERN-pane-text-injection.md`, `PATTERN.md`:** fixed with my own change to the proposed wording.
  - Done as proposed: the guarded picker dismissal is a second bounded key exception beside the guarded picker close, with its guards (the engine names the keys, a known picker with no input box, no running turn, no operator active in the quiet window, once per send attempt, an interrupt settle after the key), it is on the explicit-key-deliveries list, the guarded-interrupt opener counts the two picker paths, its pinning tests are listed, and the `PATTERN.md` line names the dismissal.
  - Not made as proposed: the `never closes a menu the operator has open` line was not reworded to say a quiet-window guard protects every open menu.
    The guarded rewind-picker close checks no operator activity (`closeRewindPicker` in `interruptguard.go`), so that sentence would be false for it.
    Line 3 now says the picker dismissal holds while an operator was active in the pane within `operator_quiet_s`, which is what protects an operator's open menu.
  - Twin claim found by the self-check: the `interruptguard.go` header said every interrupt key passes that file's guard; it now excepts the guarded dismissal.
- **F2 (MEDIUM), `fakes_test.go`, `dismisspicker_test.go`:** `guardedDismissPickerEngine` (a dismiss-picker fake that is also an `InterruptGuard`) and `TestDismissPicker_Guards`.
  - A picker over a running turn gets no key, a hold, and `picker.tried` stays unset.
  - A picker over a quiet pane gets one Escape, records the try and advances the clock by exactly the interrupt settle.
  - Placed in a new file so the fix commits stay file-disjoint; it calls `dismissPicker` directly because the guard and settle are not reachable otherwise through one tick of the gate.
- **F3 (MEDIUM), `gate_noinputbox_test.go`:** `TestGate_NoInputBoxBoundCountsHoldsInARow`, through the public `Wait` path on the fake clock.
  - Rows: a hold of another kind between no-input-box holds, a new arrival, a delivered re-prompt followed by an arrival, and a turn start after the boundary.
  - Mutation check, run and reverted before the commits: removing the else-branch reset (`wait.go`), the arrival reset (`wait.go:347`) and the `boundaryIdle` reset (`send.go`) failed the first, second and fourth row respectively.
  - Limit: the reset on a delivered re-prompt (`wait.go:582`) is shadowed by the arrival reset, since a second re-prompt needs an arrival.
    The third row pins the two together and says so in its name and the test doc comment; no row can pin `wait.go:582` alone.
- **F4 (LOW), `send.go`, `gate_test.go`:** `busyHold` has a `noInputBox` field; `dismissPicker` sets it on a failed key, and `awaitIdleSession` sets `SessionBusyError.NoInputBox` from the field or the reason constant. The active-operator hold stays unbounded.
  - Tests: a `TestGate_RepromptOverPicker` row for a key that cannot be sent (bound after `maxNoInputBoxReprompts` sends, one key per send), and `TestAwaitIdleSession_FailedDismissalKeyHoldsAsNoInputBox`.
  - Marked: the review's premise holds only when the send's deadline cuts the wait at its first poll.
    Otherwise a later poll reads the plain no-input-box reason and the count already advances, so the gate row passes without the fix.
    The unit test sets the deadline to now and fails without the fix.
- **F5 (LOW), `conditionedsend_test.go`:** the "no input box" row now asserts `NoInputBox` true, and the other rows assert it false. The assignment in `conditionedsend.go` stays.
- **F6 (LOW), `testdata/pane-autocompact-picker.txt`:** the rule line uses the `─` glyph. The wording lines are kept. `TestDismissiblePicker` still reads it as a picker, now because no rule sits below the options.
  - Not a live capture: the fixture is still drawn from the board entry's wording, and no live capture was available.

## Self-check

- fix-consistency, MEDIUM: PATTERN line 3 is false for the rewind close. Fixed with my own change (scoped to the dismissal), as under F1.
- fix-proposal, MEDIUM: same finding as above. Merged.
- fix-code-correctness, LOW: same finding, as the PATTERN line 3 scope. Merged.
- fix-consistency, LOW: the opener lost its "two". Fixed: it names the guarded interrupt and its two guarded picker paths.
- fix-affected-set, LOW: the `interruptguard.go` header twin claim. Fixed.
- fix-code-correctness, LOW and fix-proposal, LOW: the delivered-re-prompt row pins the pair of resets, not `wait.go:582` alone. Fixed by rewording the row name and the test doc comment.
- fix-proposal, LOW: the gate-loop bound sentence in the PATTERN dismissal bullet does not belong in a key-injection rule. Fixed: removed.

All four forks returned a report.

## Gate repair

The gate found the doc comment of `TestGate_NoInputBoxBoundCountsHoldsInARow` wrapped mid-sentence (`gate_noinputbox_test.go:22`), so no test ran.
The comment is now split into two sentences, one per line, with no other change.

## Disputed

None.

## Deferred

None.
