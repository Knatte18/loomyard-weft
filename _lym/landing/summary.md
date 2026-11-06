# webster begin-batch: name a delete-before-reference refusal plan_order

`lym webster begin-batch` refused a plan whose card deletes a symbol that a later card's Edit code still references, and classified the refusal `plan_conflict: tree_moved`.
That is a plan-ordering defect, not a tree that moved, so the classification sent the reader to the wrong cause.
The refusal now carries a third cause, `plan_order`, and a way forward that says to move the delete.

Closes #68

## What changes

- `websterengine.BeginBatch` classifies a refusal as `plan_order` when every blocking finding is `delete-before-reference`, whether or not a batch is begun.
  A set holding any other blocking finding keeps `plan_defect` (no batch begun) or `tree_moved` (a batch begun), since moving a delete would not settle it.
  An informational `delete-before-reference` does not count.
- For `plan_order`, `PlanConflictError.Error` replaces "edit the plan so the named cards match the tree" with "move each named delete to a card after the card that still references it".
  The `rebaseline --card NN` and re-run steps are unchanged.
- `planindex.CheckDeleteBeforeReference` names the check string once.
  `planglyph` and its tests use it, and `BeginBatch` compares against it.
- The webster master stencil lists `plan_order` among the causes.
  It ends the run stuck at once on a `plan_order` refusal, quoting the message, instead of running `lym webster fix-plan`.
  Moving a delete reshapes cards, which the plan fixer cannot do.
  The plan-fix stencil header and `websterengine`, `webstercli` and `planglyph` docs say the same.
- `loom-plan-checks-spec.md` and the package docs state that a plan gate reads the tree as it stands.
  A reference an earlier card writes during the run, or one inside an Edit target that is still a handle, is first reported by the first `begin-batch` after it lands.

## Design decisions

- The issue also suggested running the delete-before-reference check when a plan is accepted or rebaselined.
  This change does not add that.
  The plan gates already run it against the current tree.
  The conflicts that slip through depend on code an earlier card writes, which no gate can read before the run reaches it.
  Every `begin-batch` re-checks all pending cards, so a rebaseline needs no check of its own: the next `begin-batch` runs the same one before any fork.
- `plan_order` is decided per refusal from the whole blocking set, so a mixed set never loses the "match the tree" way forward.

## To check

- `plan_order` skips the plan fixer in the master stencil (`contracts/stencils/webster/webster-template-master.md`).
  Check that no other stencil, doc or caller still routes every `plan_conflict` through `lym webster fix-plan`.
- The master stencil no longer quotes the old "edit the plan so the named cards match the tree" wording in the record-batch section.
  Check that no other stencil or doc still quotes it for a refusal that can now be `plan_order`.
