# The card gate runs over the card own packages and its fixer stops at the failed check

`card-gate-own-packages` · Task · enhancement, gate, webster, weaver

## Seen

`docs/research/webster/gate-time-2026-11-28.md` ranks three card-gate levers among its top five, minutes saved per implementer row:

- Lever C, 9 to 14 min: the card gate's listed run now covers the card's packages and their dependents, 20.6 min per row against 2.0 before the dependent scope.
  Of 28 failed card tier0 runs, 14 failed only in dependents, which the batch gate also runs.
- Lever B, about 11 min: the card gate's cold fixer (`contracts/stencils/gatefix/gatefix-body-card.md`) re-runs the failed check and then the whole card gate call,
  and the next card gate call, which judges the fix, runs it in full again.
- Lever 1, 3 to 8 min: the card gate's guard step (8.1 of 28.7 card gate min per row) repeats what the batch gate runs.
  Under lever C the guard step would take back about 4.4 min, since the listed run covers fewer guard tests.

## Change

For Webster and Weaver alike:

- The card gate's listed run covers the card's own packages only; dependents run at the batch gate.
- The card gate's fixer re-runs only the check that failed, and leaves the whole card gate to the call that judges the fix.
- The guard step runs only in the batch gate, if the measured trade still holds once the two changes above are in; the run decides and says why in its change description.

The plan-level verify keeps running tier1 over everything.
