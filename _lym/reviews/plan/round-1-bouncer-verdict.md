---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (card 8 needs darn keys that card 11 adds) and gating design findings F4 and F8 at MEDIUM; no relabels."
---

The latest round holds one BLOCKING finding and two gating-class `design` findings at MEDIUM, so the decision rule gives `CONTINUE`.

- F1 (BLOCKING, design): card 8's builder refuses darn until card 11 gives darn the contract and `verify_source` keys.
- F4 (MEDIUM, design): `perchHardCap` ignores `tapestry.yaml` for the plan perch.
- F8 (MEDIUM, design): the Not-committed way forward fails for a darn run parked at Darn-Write.

The other findings (F2, F3, F5, F6, F7, F9 to F13) are non-gating.
F9 cites a decision-record departure, not a declared PATTERN entry, so it gets no special weight.
No finding was relabelled, and the review has no `## Focus departures` section to ratify or reject.
