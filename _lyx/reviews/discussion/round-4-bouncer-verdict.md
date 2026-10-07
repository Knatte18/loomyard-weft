---
verdict: CIRCLING
rationale: "F1 (design, MEDIUM) sits on d6-resume-partial-or-mixed-round, open in rounds 1, 2 and 3 and still open in round 4, so the gating key recurs; no BLOCKING finding and no focus departures."
---

Round 4 carries one gating finding: F1, class `design`, MEDIUM.
It maps to `d6-resume-partial-or-mixed-round`, which the facts list as open in rounds 1, 2 and 3.
Each round has fixed one facet of D6's resume branch, and the next review found another unspecified branch: the happy path only, the failure branches, the tamper comparison and the neither-live case.
The method gap is the same each time: D6's resume branch is specified case by case instead of by one rule that carries the fresh-round rules over.

The other findings are not gating:
- F2 (`decision`, MEDIUM) is new: failed removal of a stopped half.
- F3 (`decision`, LOW) is new: resume restarts a live fixer.
- F4 (`design`, LOW) is below MEDIUM, so it does not gate.
- F5 (`consistency`, LOW) is the `marker-standalone-path-and-cleanup` key.
- F6 (`consistency`, NIT) is new.

No finding is relabelled.
The review has no `## Focus departures` entries (it says `None`), so there is nothing to ratify or reject.
