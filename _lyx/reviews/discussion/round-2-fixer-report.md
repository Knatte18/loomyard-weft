# Fixer report — discussion round 2

## Changes

- F1 (BLOCKING, consistency): fixed as proposed.
  The parent decision is now D8, marked authoritative.
  Scope In gains picker detection with the closing Esc and the busy pre-check removal; Scope Out's picker bullet is replaced by one excluding every other dialog.
  D6 no longer sends no Esc or skips picker detection: its only Esc is D8's close, and a picker withholds the resume line.
  D7's PATTERN entry names the guarded picker close as the one exception with its own guard.
  The pre-change-entry Acceptance criterion now expects D8's outcome.
  `support-log.md`: the rejected alternative is removed, a Turn 4 records the parent override, and Q4 is marked overridden with the new answer.
- F2 (BLOCKING, design): fixed as proposed.
  D8 states the `claudeengine` picker capability reached provider-neutrally, a guarded picker close separate from the guarded interrupt, its bound (one Esc per detection, only on a picker capture, no selection, nothing more when the re-check shows no input box), a withheld resume line, and each surface's report: pause `refused`, orch watcher Warn, resume failure, shuttle send and interrupt failure.
  New sentinel `ErrRewindPickerOpen`.
  Acceptance criteria name detection, one Esc, the re-check, the failure, nothing typed, and the re-check-fails case.
  Also added: D5's `refused` covers the picker, the provider-seam constraint names the picker reading, an Open risk for a reworded picker heading, and a plan note that the picker fixture needs a real capture.
- F3 (MEDIUM, design): fixed as proposed.
  D7's Inject refusal matches any input list containing a key of the interrupt sequence, whatever its `SettleMS` or surrounding keys; the dialog-dismissal carve-out is gone and the bound reads that Inject delivers every key but the interrupt key.
  The Inject criterion has rows for a lone Escape, Escape with `SettleMS`, two Escapes and Escape among other keys.
- F4 (MEDIUM, consistency): fixed as proposed.
  D1 drops resume's pre-send check from the predicate's uses and states that resume removes its `Pair.Busy` pre-check, leaving the guarded send's readiness, which applies the turn-start hold, to decide.
  The third Open risk notes this, and the `no_running_turn` criterion adds the released-hold case.
- F5 (LOW, design): fixed with the first option the fix key offers.
  D3's Bound states that a capture quoting the hint reads as a running turn and lets an Esc onto that idle prompt.
  It also corrects the proposed wording "at most one per settle time": that holds for lyx's own retries (D4) but not for two operator-run `lyx shuttle interrupt` calls inside the settle time, so the Bound says such a pair can open the picker and D8 closes it on the next send or interrupt.

## Disputed

None.

## Deferred

None.
