# beam merge-parent's merge-in task lands as a squash, which drops the parent-branch merge

`merge-in-lands-as-squash` · Note · bug, pawl, landing, darn

Imported from [issue #66](https://github.com/Knatte18/loomyard/issues/66).

## What happened

`lym beam merge-parent knut-rheology` filed the darn `knut-rheology-merge-in`.
Its branch held a true merge commit, 5f9167ab, whose second parent was main's fbc1cf13.
The task landed into the beam through its pull request (#275 in NORCE-DrillingAndWells/Models) as a squash: the beam's new commit 628b06dd has the single parent 17010e3b.
The second parent was lost, so `git merge-base --is-ancestor fbc1cf13 knut-rheology` is still false.
The beam's pull request into main, opened next with `lym beam pr open knut-rheology` (#276), reports `dirty: the pull request conflicts with its base branch`, the same six conflicts the merge-in had resolved.
The merge-in task did nothing it existed for.

## Expected

The merge-in task lands into the beam as a merge that keeps the parent branch's commit as a parent, never as a squash.
Together with Knatte18/loomyard#65 (land it without a pull request), the simplest form is a fast-forward or true merge of the task branch into the beam.

Related: Knatte18/loomyard#64 and #65.
