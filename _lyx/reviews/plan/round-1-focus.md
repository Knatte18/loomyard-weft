---
round: 1
focus:
  - "Fidelity: check that every Decision and Constraint in `_lyx/discussion/decision-record.md` is carried by some card, and that no card adds scope the record does not license; cards 12, 17, 18 and 23 are the likeliest to carry extra scope."
  - "Attack-surface: cards 12 (agent deny), 13 (plan-gate refusal), 17 (`lyx config` write refusal) and 18 (landing hub-wide) add or change guards; settle whether each `**Intent:**` states what it lets through and what bounds it."
  - "Card order: cards 5–9 and 14 use `gateslot`, `gate` config and `hubgeom.GateSlots` from cards 1, 2 and 5; cards 20–22 use the record codec of card 19; settle whether any card depends on a later one."
  - "Verify coverage: every package the cards target, including `internal/gatecli`, `internal/gateslot` and any nested module, is run by `## verify:`, and the section follows the template's order."
  - "Edit-card glyphs: for `Edit` cards with Go-file targets whose `Intent` names symbols (cards 3, 4, 6–9, 11, 14, 15, 20–22), settle whether member glyphs and re-sign arrows are present where parameters, results or receivers change."
  - "Test economy and granularity: settle whether cards 1, 10 and 15 plan new tests for behavior an existing test already covers, and whether cards 19 and 20 or 21 and 22 split one reviewable unit too finely."
---

Seed focus only; each entry names a place and a question, with no verdict pre-stated.
