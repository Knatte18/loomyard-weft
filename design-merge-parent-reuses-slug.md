# beam merge-parent reuses a completed merge-in slug, so dobby refuses to start the second merge-in

`merge-parent-reuses-slug` · Task · bug, dobby, workspaces · running · Worktree-Create

Imported from [issue #128](https://github.com/Knatte18/loomyard/issues/128).

/home/AD.NORCERESEARCH.NO/hanf/Code/NORCE-DrillingAndWells/DrillOpPlan-LYMHUB/adopt-models-sim/.scratch/selfreport-merge-parent-slug.md

## Issue comment, 2026-11-13

Operator's expectation: a run's directory `_lym/shed/<slug>/` should be cleaned up when the run finishes (after Land and Worktree-Teardown). Here the first merge-in's run directory from 2026-11-10 was still in the prime with state `done`, which is what made batten refuse the slug. So the fix is that teardown removes the finished run's directory (and its task branch), not only that merge-parent picks a fresh slug.