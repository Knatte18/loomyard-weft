---
verdict: CONTINUE
rationale: "No BLOCKING finding, but F1 (write scope), F2 (fork-only bound) and F3 (relaunched writer) are design MEDIUM findings that map to keys open in round 1 (marker-write-vs-host-write-scope, d6-bound-covers-record-only-and-nested-forks, relaunched-writer-lost-forks-undefined), so the decision rule gives CONTINUE; no relabels, no focus departures."
---

Round 2 carries four findings, none BLOCKING.
F1, F2 and F3 are `design` at MEDIUM and each reopens a round 1 key, so they gate.
F4 is a `consistency` LOW on the poll key and does not gate.
The round 1 keys on D3's marker list, D9's provenance and the missing criteria are not raised again and are recorded as resolved; the round 2 review's focus verdicts confirm the marker vocabulary is not a defect.
No finding was relabeled, and the review has no `## Focus departures` section to ratify or reject.
