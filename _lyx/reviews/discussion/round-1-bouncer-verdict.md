---
verdict: CONTINUE
rationale: "Round 1 carries three BLOCKING design findings (F1, F2, F3) and one MEDIUM design finding (F4), all gating; F5 is a consistency finding at MEDIUM and non-gating. No relabels, no focus departures, no prior ledger."
---

The latest review has three BLOCKING `design` findings: F1 (advisor start failure told to a chair that does not exist yet), F2 (D7's `Send` needs an idle chair), F3 (a done advisor loses its strand).
F4 is a MEDIUM `design` finding (unbounded advisor-to-parent edge).
The decision rule says `CONTINUE` while the latest round carries a BLOCKING finding or a gating-class finding at MEDIUM or worse.

F5 (MEDIUM), F6, F7 and F8 (LOW) are `consistency` findings and not gating.
F6 to F8 stay worth fixing.
No finding was relabeled, and the review carries no `## Focus departures` section.
No finding cites a PATTERN entry as violated, so none gets special weight from the constraints list.
Round 1 has no earlier ledger, so no key recurs or was open earlier.
