---
round: 2
focus:
  - "Test deletions in the plan cards: for every test a card deletes on the claim that another test covers the behavior, check that the named covering test actually calls the same public surface (for example the `Runner` methods in the shuttle engine) rather than the underlying capability directly, and that a refusal-only remainder is not presented as full coverage."
  - "Stencil rewrite cards: where a card orders a check of the rest of a stencil against the recipes, check that the card states the outcome for each remaining section, so a writer knows which sections change, and that every fact the card changes has a pinned claim."
  - "Table-fold cards: where a card folds several tests into one table, check that the card states the row shape whenever the folded tests differ in setup, steps or assertions."
---

Focus targets the places where a card's stated justification may not match the code it touches.
