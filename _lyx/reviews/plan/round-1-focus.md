---
round: 1
focus:
  - "Attack-surface and refusal reachability, cards 4, 5, 8 and 9: for each new refusal (`ErrNoRunningTurn`, `ErrRewindPickerOpen`, Inject's interrupt-key refusal) and each guard that skips or lets an Esc through, does the card's `**Intent:**` state what it can skip and what bounds it, and does a named input and test reach the branch?"
  - "Fidelity to the answer key: is every Decision and Constraint in `_lyx/discussion/decision-record.md` carried by some card, and does any card (for example the PATTERN entry in card 5 or the watcher change in card 9) add scope the record does not license?"
  - "Card order and verify coverage: does any card use a symbol a later card declares (cards 1 to 3 against 4 to 9), and does `## verify:` run the tests of every package a card targets, in the template's order?"
  - "`Edit` card targets: do cards that name specific symbols in `Intent` list member glyphs, with the re-sign arrow wherever parameters, results or receivers change (for example `SessionBusyError` in card 6, `Runner` members in cards 2 and 4)?"
  - "Test economy and granularity: do cards 4, 7 and 8 plan new tests for behavior an existing shuttle, pause or CLI test already covers, or one test per symbol, and is any card split finer than an independently reviewable unit?"
---

Seed focus from the plan overview only; the card files were not reviewed.
