---
verdict: CONTINUE
rationale: "Round 1 carries F1, a MEDIUM design finding, which is gating; no relabel was made."
---

F1 says no test pins the hub wiring of the `FilingMarked` seam in `internal/battencli/startguard.go`.
A missing line or a wrong run ID or location would pass the suite and bring the stuck filed run back.
Its class is `design` and its severity MEDIUM, so the decision rule gives `CONTINUE`.
The review has no `## Focus departures` section, so there is nothing to ratify or reject.
No finding was relabeled.
This is the first round: no earlier ledger exists and no key recurs.
The fix is a local test addition whose content the finding text fixes, so it is recorded as `mechanical`.
