# Round 2 fixer report

Every finding was fixed in `_lyx/discussion/decision-record.md`; `support-log.md` gained one rejected alternative.
`lyx loom validate-discussion` exits clean.

## Changes

- F1: D5 releases the fixer only after the reviewer's done outcome, a strict parse of the review file and the fork audit, and says why the passed gate is not enough.
  D6 drops the "exhausted gate returns done with a failing gate" claim: an unparseable review after the budget is the strict-parse error, and only the fixer's caller gates can end a done run with a failing gate.
  D7 makes `Gate` the fixer's outcome and parses the review once, before the marker.
  The marker and failure criteria follow.
- F2: D1 replaces "no shuttle capability is added" with a told strand remover on `burlerengine` and the rule that the round returns the failing half's outcome; Scope lists it.
- F3: D5 has `Engine.Run` join each half's waiting goroutine before it returns, so no marker write outlives its attempt; the failure criterion drops "without waiting for the reviewer to finish" for "to finish its review".
  The rejected alternative (an attempt token in the marker) is in the support log.
- F4: D6 resume classifies each half without waiting, attaches to a both-live round with concurrent waits, and removes any marker found there; Scope names the non-waiting probe as the one shuttle addition, and the plan-writer note and resume criterion follow.
- F5: D5 gives the standalone marker location, one derivation function for both callers, and moves removal of a pre-existing marker into `Engine.Run`; the two constraints and a standalone criterion follow.
- F6: D4 states that the fixer never writes the review file, with a matching criterion.
- F7: D8 has the prior-rounds block name disputed findings beside deferred ones and says nothing else reads a dispute; Scope and the `Disputed` criterion follow.
- F8: D1 says the first failure of either half decides the round, and otherwise the fixer's run completes it.

## Deferred

None.
