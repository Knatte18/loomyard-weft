---
verdict: CONTINUE
rationale: "Round 1 carries two BLOCKING findings (F1 executable blobs skipped in WriteTreeAtRevision, F2 new hard-coded dotnet scoper branches against PATTERN-code-conventions) and two gating design findings at MEDIUM or worse (F3, F5); no relabels."
---

F1 and F2 are BLOCKING and never downgraded.
F3 (design, MEDIUM) and F5 (design, MEDIUM) are gating under the decision rule.
F4 is class scope and F10, F12, F13 are class consistency, so none of them gates.
F6 to F9 and F11 are design at LOW and do not gate on their own.
This is the first round: no prior ledger exists and no key recurs, so no departure from the review's classes or severities was made.
The review has no `## Focus departures` section.
