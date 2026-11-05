# beam merge-parent task is told to run lym fabric merge-in, which the task session refuses

`merge-in-writer-refused` · Note · bug, pawl, fabric, darn

Imported from [issue #64](https://github.com/Knatte18/loomyard/issues/64).

## What happened

`lym beam merge-parent knut-rheology` filed the darn task `knut-rheology-merge-in`.
Its body, and the verb's own help, tell the task's writer to run `lym fabric merge-in <parent branch>` in the task's own worktree.
From the task session `mo:knut-rheology-merge-in:darn`, lym refuses that verb:

> lym fabric merge-in is refused from the task session mo:knut-rheology-merge-in:darn ... when the work itself needs this verb, stop and report status: FAILED so the run ends stuck and the orch runs it

The writer could not do the one thing its task exists for.
It asked its parent orch by creel whether to fail the run.
The orch ran `lym fabric merge-in main` in the task worktree by hand, which left the pair mid-merge with six conflicts, and told the writer to resolve them, stage them and run `lym fabric merge --continue`.

## Expected

The merge-in task's writer may run `lym fabric merge-in <parent branch>` (and `merge-stage`, `merge --continue`, `merge --abort`) in its own worktree.
Alternatively, the run performs the merge-in itself before Darn-Write, and the body tells the writer only to settle the conflicts and the gate.

## Where

- `lym beam merge-parent`: the task body and help that name `lym fabric merge-in`.
- The task-session refusal of `lym fabric merge-in`, which makes no exception for the merge-in task.


## Issue comment, 2026-11-05

Same refusal hits `lym fabric merge-stage` (and presumably `lym fabric merge --continue`) from the merge-in task session: the writer resolved the conflicts but could not stage or commit them, so the orch had to run both by hand.