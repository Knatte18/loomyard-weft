# Support log — tapestry-rename

## Interview

A darn run holds no interview: the writer reads the board entry and a coarse reading of the code and decides each open question itself.
The board, its notes and `lym loom peers` showed nothing that overtakes the entry; the follow-up task `run-verbs-shared` depends on it and lists what each variant keeps.

## Rejected alternatives

- Running `tools/wordswap` over the tree with a skip list: it swaps every `loom`, and the general run's names outnumber the variant's, so the skip list would be the whole rename written inside out.
- Renaming `loom.yaml` to `tapestry.yaml`: darn and pawl read its Code-Review, driver and friction keys, so the file is the general run's; splitting its keys is design work for `run-verbs-shared`.
- Renaming the stencil files and spec files under the variant's name: their prefix is the `loom` module's, several serve darn as well, and a rename would leave stale board copies that need a removal step; their text changes instead.
- A one-time rewrite of the board, the profiles and the run records by `hubreconcile`: it needs a walk, a lock and a commit per hub, and profiles.yaml is operator-owned and never rewritten by a reconcile; a read-side alias serves every hub with no write.
- Having the program write the board worktree for this hub: a run edits only its own worktree, and another hub needs lym's own migration anyway.
- Renaming the `LOOM` program and driver strand: both serve every recipe.

## Review rounds

_No rounds yet._

## Question ledger

- Which parts are the variant's? Self-pick: what only the tapestry recipe uses and whose name says loom (D1).
- What does an empty `recipe` mean? Self-pick: the default variant, `tapestry` (D4).
- Do the strand names change? Self-pick: no, they are general (D5).
- How does another hub pick the rename up? Self-pick: read-side aliases for the old values (D3).
- Is `loom` still accepted as new input? Self-pick: no (D6).
