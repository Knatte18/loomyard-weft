---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) and F2 (MEDIUM, design) both sit on d16-d17-dropped-landing-notice-no-recovery, open in rounds 1-3 and still open: the dropped-notice recovery in D16/D17 keeps failing on a new baseline each round (unbounded ready-task start in F1, wrong board state in F2); F3 (LOW, consistency) is the d11 key and not gating."
---

F1 is a BLOCKING `design` finding on D17's second dropped-notice check.
It maps to `d16-d17-dropped-landing-notice-no-recovery`, which the facts list as open in rounds 1, 2 and 3.
F2 is a MEDIUM `design` finding on the same D16/D17 recovery path and maps to the same key.
Both are gating, so the rule gives `CIRCLING`.

F3 is a LOW `consistency` finding and maps to `d11-per-site-verdict-finish-line`.
It is not gating and does not change the verdict.

No finding is relabelled; no departures from the focus file were reported in the review.
