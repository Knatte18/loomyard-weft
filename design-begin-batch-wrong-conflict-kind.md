# webster begin-batch reports a delete-before-reference plan ordering finding as plan_conflict: tree_moved

`begin-batch-wrong-conflict-kind` · Task · bug, webster

Imported from [issue #68](https://github.com/Knatte18/loomyard/issues/68).

**What happened**
`lym webster begin-batch 20` was refused. The refusal text names a blocking finding `delete-before-reference/22-driftcli-retire-verbs`: card 22 deletes a symbol that a later card (26) still references. The way forward it gives is to edit the plan, run `lym webster rebaseline --card NN` for each edited card, then run begin-batch again.
The refusal note is classified `plan_conflict: tree_moved`.

**Why it is abnormal**
- The finding is a plan-internal ordering defect (a delete card placed before a card that still references the symbol), not a tree that moved. The `tree_moved` classification points the reader at the wrong cause.
- The ordering defect exists in the plan from the start. It surfaced only at begin-batch of batch 20, after earlier batches had run, so plan validation or earlier re-resolution did not catch it.

**What to do**
- Check why a `delete-before-reference` finding is classified `tree_moved` and give it a classification that names a plan-ordering conflict.
- Consider running the delete-before-reference check when the plan is accepted or rebaselined, so the conflict is refused before earlier batches are spent.

Provenance: task slug `drift-rework`; note file `webster-refusal-begin-batch.md`.