---
verdict: CONTINUE
rationale: "Round 2 carries eight BLOCKING findings (F1-F8), one gating-class (F8, design) and seven scope; the decision rule says CONTINUE while the latest round carries any BLOCKING finding. No key was open in an earlier round, and no focus departures or relabels apply."
---

The eight findings match the eight failing tests in the Publish failure section, so each is a real blocker.
F1 to F7 are stale or racy tests that did not follow the move of strands into own windows.
F8 is class `design` because it may hide a `Resume` defect for a moved dead own-window pane.
No finding was relabeled.
