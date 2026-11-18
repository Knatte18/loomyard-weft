---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (memory.max declared as a constant and kept as a literal, against PATTERN-code-conventions) and gating design finding F4 at MEDIUM; no finding is relabelled and no focus departures exist."
---

F1 is BLOCKING and cites a PATTERN-code-conventions violation, so it stays BLOCKING.
F4 is class `design` at MEDIUM, which the decision rule treats as gating.
F2 (scope, MEDIUM), F3 (consistency, MEDIUM), F5, F6 and F7 (design, LOW) are not gating alone, but each is carried so the fixer addresses it.
The review has no `## Focus departures` section.
