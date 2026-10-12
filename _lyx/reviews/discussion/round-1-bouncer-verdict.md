---
verdict: CONTINUE
rationale: "Round 1 carries two BLOCKING design findings (F1 merge-in and moved-parent check read different parent refs, F2 parent lock held through PR-Rework) and two MEDIUM decision findings; no relabels."
---

The decision rule gives `CONTINUE` for any BLOCKING finding and for a `design` finding at MEDIUM or worse.
F1 and F2 are both BLOCKING and class `design`.
The review has no `## Focus departures` section, and no finding was relabeled.
No finding cites a PATTERN entry; the review's own PATTERN scan found no violation.
No ledger key recurs, because this is the first round.
