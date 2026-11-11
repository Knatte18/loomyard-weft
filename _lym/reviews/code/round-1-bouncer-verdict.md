---
verdict: CONTINUE
rationale: "Round 1 carries two gating findings, F2 and F3, both design class at MEDIUM; no BLOCKING finding, no relabel, no focus departures to ratify."
---

F2 (the findings file's severity, class and change are not shown to reach the report) and F3 (the caller guard's parent-admission and unresolved-parent branches are untested) are design-class MEDIUM findings, so the decision rule gives `CONTINUE`.
F1 is MEDIUM but class consistency, so it is not gating.
F4, F5, F6 and F8 are design class at LOW, below the gating threshold.
F7 is LOW consistency.
All eight are still fixed, since class never changes whether a finding is fixed.
No finding was relabeled.
The review carries no `## Focus departures` section.
No finding cites a PATTERN entry.
