---
round: 1
focus:
  - "Fidelity: check that every D1-D8 decision and every Constraint in the decision record, including the refusal-spec row, the parity background-file entry, the stencil updates and the doc.go and docs/overview.md updates, is carried by some card, and that no card adds scope the record does not license."
  - "Card order and Uses: cards 2, 3, 4 and 6 build on each other, and card 6 depends on card 5's rename; settle whether any card depends on a later one."
  - "Verify coverage: settle whether the plan's verify section runs the tests of every package a card targets, including the integration-tagged ones for lint-pattern, the round gate and the proc-reader tests, and whether the section follows the stencil's order."
  - "Test economy: cards 1, 3 and 7 plan new tests or scenarios; settle whether an existing test already covers each, per the record's PATTERN-test-economy constraint, and whether each card lists the test-budget file under Edit."
  - "Glyphs: check Edit cards that name specific symbols in Intent for member glyphs, and for the re-sign arrow where an Intent changes a signature (for example NewVerifyGate and the Env flag in cards 5 and 6)."
  - "Attack surface: card 6 changes which gate form a merge-base reader selects and skips the headroom lint in a standalone run; settle whether the card's Intent states what that can let through and what bounds it."
  - "Trim card 10: check that the card keeps every entry name, trigger, requirement and (test) mark, and how it licenses creating background files and the links whose bytes count toward the 15000."
---

Focus targets the plan's cross-card seams, since the mechanical checks already cover format.
