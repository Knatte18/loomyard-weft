---
verdict: CIRCLING
rationale: "F1 is BLOCKING (new key failed-hinted-read-never-retried), and F2 (operator-invite-never-retried) and F3 (live-hub-list-read-unbounded) are MEDIUM design findings on keys open since round 2, still open in round 3; no relabels."
---

Round 3 carries three findings, all `design`, down from 21 in round 2.

- F1 is BLOCKING and gating, so the loop cannot converge.
  It is a new gap, not the round-2 `hint-removal-not-before-read` key: the review itself says the removal before the read is correctly ordered, and the gap is what happens after a failed read.
  It is mapped to the new key `failed-hinted-read-never-retried`.
- F2 maps to `operator-invite-never-retried` (round 2).
  The round-2 fix added a retry "at the next ensure", and F2 says no event triggers that ensure for a held channel, so the same defect stays open.
- F3 maps to `live-hub-list-read-unbounded` (round 2).
  The round-2 fix bounded the failed read's effect on events and ensures, and F3 says the holder's silence is still unbounded, so the same defect stays open.

Under the decision rule a gating finding at MEDIUM or worse on a key open in an earlier round, still open, is `CIRCLING`.
The count shrank and every other round-2 key was not raised again, so the area is nearly settled;
the two repeat keys are where the fixes stopped one step short, and the orch should weigh that when deciding whether to continue the loop.

No finding was relabelled and there are no Focus departures.
