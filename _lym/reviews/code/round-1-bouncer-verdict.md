---
verdict: CONTINUE
rationale: "Round 1 carries a BLOCKING finding (F1, test-only functions in production files against PATTERN-test-only-code), so the loop has not converged."
---

F1 is BLOCKING and cites `PATTERN-test-only-code`, which the declared changes do not list, so the overview's current text binds.
The decision rule returns `CONTINUE` on any BLOCKING finding.

The review states that the record's added entry allows what the PATTERN entry forbids.
The entry wins over that added entry in this judgment.
F1's fix defers the reconciliation to the run's parent.

No finding is relabelled.
No finding is class `design`.
The review has no `## Focus departures` section, so there is no departure to ratify or reject.
