---
verdict: CONTINUE
rationale: "Round 1 carries two gating design findings at MEDIUM, F2 (in-place truncating write contradicts the writes-nothing claim and the D4 Bound) and F3 (lock placement and timing on a hub with no _board); no BLOCKING finding and no relabel."
---

The decision rule makes `design` the gating class.
F2 and F3 are both `design` at MEDIUM, so the loop continues.
F1 and F4 are MEDIUM but `consistency`, and the rest are LOW or NIT, so none of them gates.
No finding is relabeled; no departed finding to name.
The review carries no `## Focus departures` section.
This is round 1, so no fix can converge it, and there is no prior ledger.
