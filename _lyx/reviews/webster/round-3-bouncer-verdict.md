---
verdict: CONTINUE
rationale: "Round 3 carries BLOCKING finding F1 (scope), the cmd/lyx test budget row left at 73 against 74 top-level tests, which fails TestTestBudget_PackagesStayWithinBudget in Publish; the key is first raised this round, so it is not circling."
---

F1 is BLOCKING, and the decision rule says `CONTINUE` while the latest round carries any BLOCKING finding.
Its class is `scope`, so it is not gating-class, but severity alone keeps the loop open; I relabeled nothing.
F1's key, `cmd-lyx-test-budget-row-not-raised`, is absent from the facts' list of keys open in an earlier round, so `CIRCLING` does not apply.
The two keys open in round 1 were resolved in round 2 and did not return.
The review has no `## Focus departures` section.
