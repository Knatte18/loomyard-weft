---
round: 1
focus:
  - "Fidelity: check every D1–D14 Decision and every Constraint in `_lyx/discussion/decision-record.md` is carried by some card, especially D6 (session-message channel), D7 (failure and hang rules), D10 (quiet advisor turns) and D13 (test tiers); and that no card adds scope the record does not license."
  - "Card order: cards 4–7 (seatengine), 8–9 (fake runner and adapter) and 10–11 (Env fields and recipe entry) each build on earlier packages; settle whether any card's `Uses` reach a symbol a later card creates, notably card 5 and 6 against card 2 and 3 (include form and seat stencils), and card 8 against card 9's `SeatRunner` seam."
  - "Test economy: cards 4–7, 12 and 13 each plan tests over one engine; settle whether a card plans a new test for behavior an existing test or an earlier card's public-surface test already covers."
  - "Attack-surface: card 1 (`Spec.QuietHold`, per-run send lock) and the advisor-to-chair message path (D6, D7) introduce a hold that suppresses turn end and a send route; settle whether each card's `**Intent:**` states what it can skip or let through and what bounds it."
  - "Edit-card glyphs: for each `Edit` card (1, 2, 10, 11, 14 and any other), settle whether Go file targets are listed without member glyphs while `Intent` names symbols, and whether signature changes carry the re-sign arrow."
  - "Verify coverage: settle whether the `## verify:` section runs the tests of every package the cards target, including `internal/seatengine`, `internal/stencil`, shuttle, shedadapters, shedrecipe and any nested module or build-tagged tests, and is in the template's order."
---

Seed focus drawn from the overview and decision-record headings; the card files were not read in this pass.
