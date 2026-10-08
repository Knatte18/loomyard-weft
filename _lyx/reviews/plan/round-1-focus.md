---
round: 1
focus:
  - "Fidelity: check that every numbered Decision (1 to 14) and every Constraint in `_lyx/discussion/decision-record.md` is carried by some card, and that no card adds scope the record does not license."
  - "Attack surface: cards 6, 8, 9 and 10 add a hub lock, an unconditional single-pair reconcile and a safety-net `Preflight` row; settle whether each card's `**Intent:**` states what it can skip or let through and what bounds it."
  - "Card order: the shared-leaf and fabric-read cards (1 to 5) must precede their users (6 to 10); check cards 12 and 13 against 11, and 15 to 19 against 14, for a dependency on a later card."
  - "Granularity: cards 15 to 19 each wire the edit directive into a role group; settle whether each is an independently reviewable unit and whether card 7 and card 4 stand alone or belong in a neighbour."
  - "Test economy: settle whether any card plans a new test function or file where an existing stencilstore drift test or role-stencil scan already covers the behavior, as `PATTERN-test-economy` and the record's Constraints require."
  - "Member glyphs: check each `Edit` card whose Intent names specific symbols (for example cards 5, 10, 12, 13) for member glyphs, and for the re-sign arrow where the Intent changes parameters, results or receiver."
  - "Verify coverage: settle whether the `## verify:` section runs the tests of every package a card targets, including `contracts/stencils`, and follows the plan template's order."
---

Seed focus only; the reviewer still judges the whole plan against the rubric.
