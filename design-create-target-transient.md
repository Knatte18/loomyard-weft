# webster create-not-done check rejects a Create target the card removes itself

`create-target-transient` · Note · bug, webster, loom, quarry

Imported from [issue #731](https://github.com/Knatte18/loomyard/issues/731).

**What happened**

A card named a git-ignored scratch file as its one `Create:` target and also told the implementer to remove that file by its literal path once `lyx board upsert` had read it.
After the batch, `lyx webster record-batch` ran its `create-not-done` check, looked the target up through quarry, got `not_found (the unit is missing)` and refused the batch:

```
create-not-done/<card>[blocking]: Create target ".scratch/<file>.md#" still does not resolve: looked up ".scratch/<file>.md#", quarry answered not_found (the unit is missing)
way forward: lyx webster recover-batch <batch>
```

The board write itself had succeeded.
Recovery worked only by leaving the git-ignored draft in place, refilled from `lyx board get --body`, so the target resolved.

**Why it is abnormal**

The card's removal step and the check cannot both hold.
A card whose only effect lies outside the repo, such as a board write, has no target type the check accepts for a transient draft.
A planner following the cleanup rule for scratch files produces a card that Webster always rejects.

**What to do**

Give the planner a way to declare a transient or non-persistent `Create:` target that `create-not-done` skips, or exempt `.scratch/` paths from the check.
Alternatively, have the card validator reject a card that both creates and removes the same path, so the conflict surfaces at planning time rather than at `record-batch`.

Provenance: task `orch-rules-files`; notes `02-orch-rules-verb-recovery.md`, `webster-refusal-record-batch.md`.
