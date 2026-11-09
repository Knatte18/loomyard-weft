---
verdict: CONTINUE
rationale: "Round 1 carries two gating design findings at MEDIUM (F1 coverage build runs outside the module root, F2 signal recorded as a build failure); no relabels, no BLOCKING."
---

F1 and F2 are class `design` at MEDIUM, so the decision rule gives CONTINUE.
F3 and F4 are MEDIUM but class `scope`, which is not gating, and F5-F9 are LOW or NIT.
No finding was relabeled, and the review has no `## Focus departures` section.
This is round 1, so no key recurs and none was open earlier.
