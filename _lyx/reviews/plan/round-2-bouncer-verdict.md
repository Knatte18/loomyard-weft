---
verdict: CONTINUE
rationale: "Round 2 carries BLOCKING F1 (card 17 omits three existing tests the new shell-wait rule breaks), so the loop continues; the design-class MEDIUMs F3 and F4 are first raised this round and do not block on their own, and no relabel was made."
---

Round 2 raised seven findings: one BLOCKING (F1, scope), four MEDIUM (F2 consistency, F3 design, F4 design, F5 scope) and two LOW (F6 consistency, F7 design).
The decision rule continues on any BLOCKING finding, and F1 is one.

All eleven round-1 keys were not raised again, so the ledger marks them resolved.
Every round-2 finding is on a key first raised this round.
F3 and F4 are gating-class MEDIUMs on first-raised keys; they would not block alone, since Go carries them into the decision record's `## Open risks`.
F3 changes a verdict's behaviour (a gone-branch stop no longer refuses), so it is marked `design-wrong` and its fix needs a fresh review.
No finding was relabelled, and the review has no `## Focus departures` section.
