# No run step edits its own task's board entry

Batch `01-doc-lifecycle-wording` recorded `status: done` at head `3f6cd1c5d3d0b4ea20b3e996c4600dd519e3a212`.
The digest listed no deviations and no warnings, so the scope guard found no file changed outside the batch's targets.

The batch's fork committed four cards, each with its own gate green:

- Card 1: `CLAUDE.md` and `docs/overview.md` now say the building task leaves its own entry to the landing and prune.
- Card 2: the Discussion stencil and its rubric forbid and flag a decision that edits the own entry.
- Card 3: the Plan stencil and its rubric forbid such a card and route a decided own-entry step's drop through the parent.
- Card 4: the darn write stencil and its rubric forbid such a step and route its omission through the parent.

The plan-level verify command is left to the gate.
