---
verdict: CONTINUE
rationale: "Round 1 carries two gating design findings at MEDIUM (F1 card order breaks hubforge-hub tests at card 4's commit, F3 no stated bound for a failed restore in the hub-wide write); F2 is MEDIUM scope and F4 to F6 are LOW, all non-gating; no relabels."
---

The latest review has no BLOCKING finding.
Two findings are gating under the decision rule, both class `design` at MEDIUM: F1 and F3.
F2 is MEDIUM but class `scope`, so it is fixed but does not gate.
F4 (design), F5 (consistency) and F6 (scope) are LOW and do not gate.

No finding was relabeled, and the review has no `## Focus departures` section.
There is no previous ledger, so no key recurs.
