---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (uncapped prebuild go build, PATTERN-gate-slot) and design-class MEDIUM findings F2, F3 and F4; no relabels."
---

F1 cites `PATTERN-gate-slot` and stays BLOCKING.
F2, F3 and F4 are `design` at MEDIUM, so they are gating.
F4 cites `PATTERN-cwd-resolution`, which makes it gating by citation as well.
F5 and F6 (scope), F7 and F8 (consistency) at LOW, and F9 (consistency) at NIT, are not gating.

No finding was relabeled, and no focus departures were reported.
This is the first round, so no fix counts toward convergence.
