---
verdict: CONTINUE
rationale: "Round 3 carries four BLOCKING design findings (F5, F6, F7, F8), each on a new key, and all four keys left open after round 2 are resolved, so the loop has not converged and is not circling; no finding was relabelled."
---

F5 to F8 each name a defect no earlier ledger key records:

- F5: D2 matches a resource class that, by upstream source, no Konsole window has on Wayland.
- F6: a script run can reach another script, because KWin reuses ids.
- F7: the query does not report what D6 orders by.
- F8: D9's renumber rewrites the order keys of windows still waiting for placement.

F6 also disputes D3's bound on a killed child's script, which touches the subject of `timeout-or-kill-leaves-script-and-half-built-desktop`.
I keep that key resolved: F6's defect is id reuse between overlapping runs, which needs no timeout or kill.
F8 concerns D9's renumber, as `renumber-collides-with-absent-order-keys` and `overflow-overwrites-remembered-desktop` do,
but its case is a reopened window with an unsettled entry, which neither key records.

The review's `prior` list resolves F1 to F4, which closes the keys open after round 2.
The review has no `## Focus departures` section.
Its `## Rejected` items are not findings and carry no ledger key.
No PATTERN entry is cited by any of the four findings.
