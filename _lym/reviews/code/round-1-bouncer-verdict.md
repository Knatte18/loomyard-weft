---
verdict: CONTINUE
rationale: "Round 1 carries four design-class MEDIUM findings (F1 scope that never starts reads as a failing test, F2 unbounded probe, F5 untested out-of-memory branch of RunAlone, F6 untested capped report-step and testmap paths), so the gating rule says CONTINUE; no relabels, no Focus departures section."
---

Round 1 has no BLOCKING finding.
The gating findings are F1, F2, F5 and F6: each is class `design` at MEDIUM.
F3 (decision), F4 (scope) and F7, F8 (consistency) are not gating; F9 and F10 are design at LOW, below the gate.
F3 is deferred to the run's parent, as the review's fix text says.
No finding was relabeled and the review has no `## Focus departures` section to ratify or reject.
The review's two rejected candidates are not findings and do not enter the ledger.
