---
round: 1
focus:
  - "Cards 1-2: check that batch-boundary logic in the cost batchifier decides ref coverage only through `planparser.RefCovers`, `RefScope` or `RefFile`, and that a recorded `state.json` partition is never regrouped."
  - "Card 2: check that a rename or delete of an earlier candidate-batch target starts a new batch in both directions (rename of a target, delete of a target), and that unrelated cards still batch together."
  - "Cards 3 and 5: check that the exported impactset guard scan and command limit are the ones webster uses, with no duplicated marker scan, and that a walk that cannot read a path is refused rather than silently skipped."
  - "Cards 4 and 6: check that guard-step wording lives in webster's stencils with Go filling markers only, and that every card gate ends with the root-module guard step, falling back to a prompt line when the scan fails or the step is over-long."
  - "Per-card mechanical checks for every group of every card, and test economy for the new tests against existing planparser, batcher and websterengine tests."
---

Focus follows the plan's two themes: the batching rule (D1) and the guard step (D2 to D4).
