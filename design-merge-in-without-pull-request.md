# beam merge-parent's merge-in darn should land without a pull request

`merge-in-without-pull-request` · Note · enhancement, pawl, darn, landing

Imported from [issue #65](https://github.com/Knatte18/loomyard/issues/65).

## What happened

`lym beam merge-parent knut-rheology` filed the darn task `knut-rheology-merge-in`, which landed through its own pull request into the beam (#275 in NORCE-DrillingAndWells/Models).
The merge's tree was identical to the beam's, since main's commit was a squash of content the beam already carried, so the pull request had no changed files.
The operator still had to read and approve an empty pull request, and the run spent a PR-Review round and a Describe on it.

## Wanted

The merge-in darn that `lym beam merge-parent` files needs no pull request.
It lands straight into the beam once its gate and its Code-Review of the merge pass, with no PR-Review row and no GitHub pull request, whether or not the merge changed files.
The merge is reviewed again anyway, as part of the beam's own pull request into its parent.

Related: Knatte18/loomyard#64, where the same task's writer is refused the merge verbs it needs.
