# Round 2 fixer report

## Fixed

- F1 (MEDIUM, decision): D3 now states that a check whose activity stamp differs from the previous check's drops the accumulation and does not add its own gap, so the accumulation only holds gaps from intervals that begin after the stamp.
  This keeps D3's existing claim that the accumulation holds only gaps after the newest activity.
  AC9 gains a row: a check that observes both a sleep gap and a changed stamp subtracts that gap from no later check.
- F2 (LOW, consistency): the first Constraints bullet no longer cites `PATTERN-transient-stop` and `PATTERN-shed-verb-set`, which govern the transient mark and the shed verb set.
  It now states the constraint directly: `BudgetExempt` stays a field of `shedengine.OutputPointer`, and D1 sets it only on the Burler adapter's two hand-back returns, beside the Bouncer's existing budget-continue exemption in `settleUnconverged`.
  The false "never on a verdict" clause is gone.

## Disputed

None.

## Deferred

None.
