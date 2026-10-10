---
round: 1
focus:
  - "Fidelity: check that each of decision-record D1-D13 and every Constraint is carried by some card, in particular D8 (bounded verify-log history), D9 (Finalize commits parent run records before its parent-side merge), D10 (teardown board-claim settle on landed and abandoned runs) and D13 (refusal-spec rows); also check that no card adds scope the record does not license."
  - "Card order: cards 6, 11 and 12 share the verify-gate decision and cards 8-12 build the recipe up from config to routing; settle whether any card uses a symbol or file a later card creates or changes."
  - "Attack surface: cards 6 (told command, optional merge base), 12 (recipe-selected arms, reject rework row) and 13 (batten admits `darn`) add routing edges and a verify bypass form; settle whether each card's Intent states what the new path can skip or let through and what bounds it, including the budget floor of 1 in card 11 and the halt when the budget is spent."
  - "Granularity and test economy: cards 1-5 and 8-11 each sit near existing tests for the verify gate, Finalize, teardown, halt note and recipe loading; settle whether any card plans a new test function or file for behavior an existing test already covers, naming that test."
  - "Edit cards: for each `Edit` card whose targets are only non-test Go file paths while its Intent names specific symbols, settle whether member glyphs are missing; for each member glyph whose Intent changes parameters, results or receiver, settle whether the re-sign arrow is present."
  - "Verify coverage: settle whether every package the cards target, including any nested module or build-tagged tests, is run by the `## verify:` commands and whether the section follows the plan template's order."
---

Seed focus drawn from the plan overview, the card index and the decision-record headings; no card bodies were read in depth.
