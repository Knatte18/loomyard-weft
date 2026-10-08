---
verdict: CONTINUE
rationale: "F1 is a MEDIUM design finding (a failed reconcile or commit step leaves written config uncommitted, and a retry stamps the build over it), and the decision rule makes any gating-class finding at MEDIUM or worse CONTINUE; no relabel applied."
---

Round 1 carries four findings: F1 (MEDIUM, design), F2 and F3 (LOW, scope), F4 (NIT, consistency).
F1 is the only gating one, and its own entry shows the class and severity are correctly labelled: the reconcile contract ("commits everything it wrote") is broken on the error path.
No finding is relabelled, and no Focus departures section exists to ratify or reject.
No ledger key recurs and none is open from an earlier round, as this is the first round.
