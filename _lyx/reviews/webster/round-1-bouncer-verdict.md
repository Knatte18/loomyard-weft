---
verdict: CONTINUE
rationale: "Round 1 carries two gating design findings at MEDIUM, F1 (wireDarn tells the verify gate the per-call scoped command) and F2 (nested trees drop the more specific tree's target), and no relabel applies."
---

The latest review has no BLOCKING finding, but F1 and F2 are `design` at MEDIUM, so the decision rule gives `CONTINUE`.
F3 is `design` at LOW and does not gate.
F4 and F5 (`scope`) and F6 and F7 (`consistency`) are non-gating.
No finding was relabeled, and the review has no `## Focus departures` section to ratify.
The facts list no recurring or earlier-open keys, and no prior ledger exists.
