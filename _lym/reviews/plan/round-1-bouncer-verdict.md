---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (tier0 test spawning sh, barred by PATTERN-test-speed) and design-class MEDIUM F2 (argument cap off by one), so the loop has not converged; no relabels."
---

The latest round holds one BLOCKING finding, F1, which cites a violation of `PATTERN-test-speed` and so stays BLOCKING.
F2 is class `design` at MEDIUM, which is gating under the decision rule.
F5 and F10 are `design` at LOW and do not gate.
F3 is `scope`, F4, F6, F8 and F9 are `consistency`, and F7 and F11 are `decision`, all LOW or MEDIUM-and-non-design, so none gate.

No finding was relabeled and the review has no `## Focus departures` section.
This is the first round, so no ledger key recurs and fixes are not yet reviewed by a fresh reviewer.
