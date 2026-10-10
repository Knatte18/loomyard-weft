---
round: 1
focus:
  - "Cards 4, 5, 6 (reed watchdog loops) and cards 11, 12 (resume-context delivery): does each `Intent` that adds a wake source, signal file, verb or skip path state what it can now skip or let through and what bounds it, as D4, D5 and D11 of the decision record do?"
  - "Card order across the shared packages: do cards 5, 6, 8, 9, 11 and 12 use symbols that a later card creates (`NextWakeCadence`, `DiscoverSignalFileName`, `ChangeDir`, the delivery mark, the readiness reading)? Name the card and the later card."
  - "Fidelity: is every Decision and Constraint in `_lyx/discussion/decision-record.md`, D1 to D11 and the Scope-In items including the docs list, carried by some card, and does any card add scope the record does not license, including from the carry-over entries in `## Open risks`?"
  - "`Edit` cards whose targets are only non-test Go file paths while `Intent` names symbols (cards 2 to 6, 8, 9, 11, 12): are member glyphs listed, with the re-sign arrow where `Intent` changes parameters, results or receiver?"
  - "Test economy in cards 2, 4, 5, 6, 8, 9, 13: does a card plan a new test for behavior an existing test or the public-surface test of the same card already covers, or one test per symbol?"
  - "Granularity of cards 1, 7, 10, 13: is a private supporting type or inseparable constructor split into its own card, or a public-surface symbol bundled away?"
  - "`## verify:` coverage: does the command set run the tests of every package any card targets, including `cmd/lyx` for the guard, `docs/` benchmark tooling, and the integration-tagged tests, and is the section in the order the plan template states?"
  - "`ImpactSummary` on `Edit` and `Delete` cards: is it a real blast-radius conclusion and not a restatement of `Intent`?"
---

Focus covers the areas where the plan's structure (ordering across `reedengine`, `shuttleengine` and `orchengine`) and the rubric's attack-surface and fidelity questions are most likely to bite.
No concrete instance is promoted: none was checked against the card files yet.
