---
verdict: CONTINUE
rationale: "Round 1 carries two design-class MEDIUM findings (F1 dropdown offers done projects that checkProjectField refuses, F2 no test pins a done project's non-root parent), both gating; F3 is a decision-class MEDIUM deferred to the parent and does not gate. No relabels."
---

Round 1 has no BLOCKING finding.
F1 and F2 are design-class at MEDIUM, so the decision rule gives `CONTINUE`.
F3 is class `decision`, a non-gating class; its fix defers to the run's parent and changes no code.
No finding was relabeled, and the review has no `## Focus departures` section.
