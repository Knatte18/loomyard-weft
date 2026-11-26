# Keep the decision record and review findings of each landed task in the weft

`keep-landed-records` · Task · enhancement, landing, fabric, report · starting · Open-Window

Imported from [issue #229](https://github.com/Knatte18/loomyard/issues/229).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `_bolt/.lym/creel/journal`.

## Seen (operator, 2026-11-24)

An analysis of the parent's PR-Review findings in the C# hubs could not finish its work, because the evidence was gone.
About 29 findings were in scratchpad files under `/tmp` that no longer exist.
The decision records (Sketches) of the landed tasks could not be read: the board holds only open entries, and the task worktrees are deleted.
So the analysis could not say whether delivered code broke its decision record, and every gate attribution was inferred.
The only complete source left was each hub's creel mail journal under `_bolt/.lym/creel/journal`.

The operator wants this kept: absolutely.

## Idea

At landing, store what a later study needs in the weft, where it survives the worktree's deletion.

- The decision record with its `Added after Discussion` entries, as it stood at landing.
- Every Code-Review round's findings and the fixer reports, the parent's rulings and answers, and the PR-Review findings.
- The Describe text as it was merged, and the drift verdict.
- A short index row per landed task, so `lym report` can read them without walking directories.
- Read first which records survive today and where, and how landing already archives a run (the archive tag, `Reconcile leftover shuttle runs from run records`).
- Decide the weft path and how long records are kept, and make sure findings no longer live only in a scratchpad directory.
