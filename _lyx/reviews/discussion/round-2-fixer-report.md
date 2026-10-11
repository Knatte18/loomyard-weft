# Fixer report: discussion, round 2

All three findings were validated against `internal/shedadapters/bouncer.go` and `internal/shedengine/run.go` and fixed.

## Fixed

- F1 [MEDIUM, design]: confirmed that a refusal's `degrade` Stuck bypasses `settleUndecided`, the only caller of `budgetReached`/`escalate`, and meets Shed's `episodeStuckCount >= effectiveMaxBounces` block.
  Decided to keep Shed's block rather than escalate, because escalating needs a judged round and a ledger pointer, and the refusal has just archived both.
  D9's Bound now states that at a reached budget one refusal blocks the run with no re-judge and no escalation, explains why, and names the block's `lyx shed goto` way forward.
  Added an Open risks bullet and an acceptance criterion pinning that exit.
  In the support log, the "Halting the run" rejected alternative now names this exception, a new rejected alternative records escalating the refusal, and Q10 records the pick.
- F2 [LOW, consistency]: D7's sentence now says no CONVERGED verdict reaches Done unchecked.
  A new line in the Bound exempts a pending escalation accept the parent recorded, bounded by the parent's decision, so the guard never refuses it.
- F3 [LOW, consistency]: D10 now states that a judge pass whose result reached no Call records nothing, naming the case where a later Call settles a crashed Call's unharvested judgment from disk (D8).
  The usage acceptance criterion now reads "one `judge` entry per judge pass whose result reached a Call".

## Disputed

None.

## Deferred

None.
