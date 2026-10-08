---
verdict: CONTINUE
rationale: "Round 1 carries two BLOCKING design findings (F1, F2) and two MEDIUM design findings (F3, F6), all gating; no relabels, no Focus departures section present."
---

The latest review has no `## Focus departures` section, so there is nothing to ratify or reject.
No finding is relabeled; every class and severity stands as the review entry gives it.
No cited finding names a `PATTERN-*` entry as violated by id, so no entry-wins conflict arises.

Gating findings (class `design`):

- F1 BLOCKING: D7 subtracts a border row the layout never loses.
- F2 BLOCKING: D12 orders the booter's own session last and leaves nested ups unbounded.
- F3 MEDIUM: D12's concurrency claim fails for a booter that finds the server up.
- F6 MEDIUM: config color values have no stated bound and no named reader.

Non-gating findings (F4, F5, F7, F8, F9) are still to be fixed; class never changes whether a finding is fixed.
No ledger key recurs, since this is the first round.
