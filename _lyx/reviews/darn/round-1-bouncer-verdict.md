---
verdict: CONTINUE
rationale: "Round 1 carries F1, a design-class MEDIUM finding (exact join under a pair-opening commit is unpinned), which is gating; F2 and F3 are LOW. No relabels."
---

F1 is class `design` at MEDIUM, so the decision rule gives `CONTINUE`.
F2 and F3 are LOW and do not gate on their own, but their fixes are local test additions and ride along with F1's.
No finding was relabeled, and the review has no `## Focus departures` section.
