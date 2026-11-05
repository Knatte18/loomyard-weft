---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (design, no refusal of a lock path beside a Bolt file, PATTERN-tracked-file-locks overstates its test) and gating MEDIUM F3 (design, shedadapters lock move untested); no relabels."
---

Round 1 is the first round, so no fix has converged and no ledger key recurs.

Gating findings: F1 (design, BLOCKING) and F3 (design, MEDIUM).
F2 (scope, MEDIUM) is not gating by class, and F4 (design, LOW) is gating by class but below MEDIUM.
F5 and F6 (consistency, LOW) are non-gating.
No finding is relabeled; F1 cites an entry the overview lists and stays BLOCKING.
The review has no `## Focus departures` section.
