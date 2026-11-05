# beam merge-parent: the merge-in task can merge, lands as a merge and needs no pull request

`merge-parent-lands-right` · Task · bug, pawl, darn, landing, fabric · draft

Depends on: [`drift-rework`](design-drift-rework.md), [`clone-reset-deletes-first`](design-clone-reset-deletes-first.md)

## From note merge-in-writer-refused

Imported from [issue #64](https://github.com/Knatte18/loomyard/issues/64).

### What happened

`lym beam merge-parent knut-rheology` filed the darn task `knut-rheology-merge-in`.
Its body, and the verb's own help, tell the task's writer to run `lym fabric merge-in <parent branch>` in the task's own worktree.
From the task session `mo:knut-rheology-merge-in:darn`, lym refuses that verb:

> lym fabric merge-in is refused from the task session mo:knut-rheology-merge-in:darn ... when the work itself needs this verb, stop and report status: FAILED so the run ends stuck and the orch runs it

The writer could not do the one thing its task exists for.
It asked its parent orch by creel whether to fail the run.
The orch ran `lym fabric merge-in main` in the task worktree by hand, which left the pair mid-merge with six conflicts, and told the writer to resolve them, stage them and run `lym fabric merge --continue`.

### Expected

The merge-in task's writer may run `lym fabric merge-in <parent branch>` (and `merge-stage`, `merge --continue`, `merge --abort`) in its own worktree.
Alternatively, the run performs the merge-in itself before Darn-Write, and the body tells the writer only to settle the conflicts and the gate.

### Where

- `lym beam merge-parent`: the task body and help that name `lym fabric merge-in`.
- The task-session refusal of `lym fabric merge-in`, which makes no exception for the merge-in task.


### Issue comment, 2026-11-05

Same refusal hits `lym fabric merge-stage` (and presumably `lym fabric merge --continue`) from the merge-in task session: the writer resolved the conflicts but could not stage or commit them, so the orch had to run both by hand.

## From note merge-in-lands-as-squash

Imported from [issue #66](https://github.com/Knatte18/loomyard/issues/66).

### What happened

`lym beam merge-parent knut-rheology` filed the darn `knut-rheology-merge-in`.
Its branch held a true merge commit, 5f9167ab, whose second parent was main's fbc1cf13.
The task landed into the beam through its pull request (#275 in NORCE-DrillingAndWells/Models) as a squash: the beam's new commit 628b06dd has the single parent 17010e3b.
The second parent was lost, so `git merge-base --is-ancestor fbc1cf13 knut-rheology` is still false.
The beam's pull request into main, opened next with `lym beam pr open knut-rheology` (#276), reports `dirty: the pull request conflicts with its base branch`, the same six conflicts the merge-in had resolved.
The merge-in task did nothing it existed for.

### Expected

The merge-in task lands into the beam as a merge that keeps the parent branch's commit as a parent, never as a squash.
Together with Knatte18/loomyard#65 (land it without a pull request), the simplest form is a fast-forward or true merge of the task branch into the beam.

Related: Knatte18/loomyard#64 and #65.


## From note merge-in-without-pull-request

Imported from [issue #65](https://github.com/Knatte18/loomyard/issues/65).

### What happened

`lym beam merge-parent knut-rheology` filed the darn task `knut-rheology-merge-in`, which landed through its own pull request into the beam (#275 in NORCE-DrillingAndWells/Models).
The merge's tree was identical to the beam's, since main's commit was a squash of content the beam already carried, so the pull request had no changed files.
The operator still had to read and approve an empty pull request, and the run spent a PR-Review round and a Describe on it.

### Wanted

The merge-in darn that `lym beam merge-parent` files needs no pull request.
It lands straight into the beam once its gate and its Code-Review of the merge pass, with no PR-Review row and no GitHub pull request, whether or not the merge changed files.
The merge is reviewed again anyway, as part of the beam's own pull request into its parent.

Related: Knatte18/loomyard#64, where the same task's writer is refused the merge verbs it needs.


