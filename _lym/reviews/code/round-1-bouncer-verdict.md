---
verdict: CONTINUE
rationale: "Round 1 carries one gating finding, F1 (design, MEDIUM, internal/landingshed/prreview.go:323), so the rule gives CONTINUE; F2 and F3 are LOW and do not gate; no relabels."
---

F1 says that with the setting on, the run does not wait at PR-Review for the orch's answer.
The operator's ending or an approval can finish the row before the orch has answered.
Its class is `design` and its severity MEDIUM, so it is gating under the decision rule.
Its own entry shows no mislabel, so I relabeled nothing.

F2 (a failed record write after a successful post re-sends the request) and F3 (the loom wiring closures lack a test) are LOW, so they do not gate.
The review has no `## Focus departures` section, so there is nothing to ratify or reject.
No ledger key recurs, and none was open in an earlier round.
