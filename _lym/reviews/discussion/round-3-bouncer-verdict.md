---
verdict: CIRCLING
rationale: "F5 is gating MEDIUM design on key counter-bursts-readded-or-reweighted, open in round 2 and still open in round 3 after the one-way raise fix; F1 and F2 are also BLOCKING design findings on new keys. No relabels."
---

Round 3 carries two BLOCKING design findings, F1 and F2, both on keys first raised in round 3.
They alone would give `CONTINUE`.

F5 changes the verdict.
Round 2 raised `counter-bursts-readded-or-reweighted` at MEDIUM design, and the fix added a one-way raise.
F5 shows the same counter still fails for a re-added account above the others' lowest and for a lowered weight, so the key is open again at gating severity.
The decision rule gives `CIRCLING` for that.

The other keys the facts list as open in an earlier round were not raised in round 3 and are resolved in the ledger.
`d6-known-directories-lose-delisted-and-landed` is among them: the review's Rejected section says D6's route is stated and the remaining gap is F3, a new key.

F1 is filed on a new key, `fresh-launch-session-sources-incomplete`, not on `search-session-ids-no-source-into-engine`.
Round 2 asked for a source of ids into the engine; F1 says the list of sources is incomplete, which is a different defect.

No finding is relabelled and the review has no `## Focus departures` section.
