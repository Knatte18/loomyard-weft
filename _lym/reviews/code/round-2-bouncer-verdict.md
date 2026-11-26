---
verdict: CONTINUE
rationale: "Round 2 carries one BLOCKING design finding, F8: the range key hashes only files that exist at the review base, so a commit touching a file the range adds neither re-judges an uncaused ruling nor drops a stale carried_by ruling."
---

The decision rule gives `CONTINUE` while the latest round carries any BLOCKING finding, and F8 is BLOCKING.

F8 is a new key, `range-key-omits-added-files`.
The seven keys open in round 1 are all marked resolved by the round-2 review, and F8 maps to none of them.

No finding is relabelled.
F8 cites no PATTERN entry.
The review carries no `## Focus departures` section, so there is no departure to ratify or reject.
