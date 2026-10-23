# Run tail: Merge-In with fixer before a Code-Perch, an optional PR block, and a locked re-merge at Finalize

`loom-tail-review` · Note · enhancement, loom, burler, shed, fabric, batten

## Design (operator, 2026-10-23)

The run's tail after the code is written.

### Loom

1. **Webster** ends only when every test passes.
2. **Merge-In with alignment and a fixer**, without the parent's merge lock.
   It merges the parent in, resolves conflicts, runs the gated tests and fixes until they pass.
   The alignment agent's check becomes part of this fixer: it reads what the merge brought in against the decision record, fixes what follows from the record, and escalates a real conflict between decisions instead of choosing one.
   The fixer is thorough: a strong agent or a twill, and one perch round when the change the merge brought in is large.
3. **Code-Perch**, a general perch over the task's code, in place of today's Webster perch, since the code is already designed in detail and tested before Webster ends.
   It runs after the merge on the bet that nothing large lands on the parent between this Merge-In and the Land block.
4. **PR block**, only when a config flag is on: it opens a PR for a human, and an interactive session takes the human's feedback and fixes it.
   Its design waits until a human PR review is needed; with the flag off the run skips it.
5. **Land block**, one producer for the whole locked operation:
   1. Take the parent's merge lock.
   2. Merge the parent in; on a conflict or a failing test, fix it.
      Run right, the parent brings nothing large here, so this is quick.
   3. Run the Describe agent, so the landing's commit message is up to date.
   4. Land on the parent (squash, push, board done).
   5. Release the lock.

   The lock covers only this block, so the parent cannot move between the fix and the landing, and the lock is held as briefly as possible (as Millhouse does it).

### Darn

A darn run is quick and narrow, and is not run beside work that can make it drift, so it has no Merge-In before its review: the writer (ending only when every test passes), Darn-Review as its Code-Perch, the PR block behind the same flag, and the same Land block.
Loom has the step 2 Merge-In because a loom run can take hours.

### Gone

From today's tail (`contracts/recipes/loom-recipe.yaml`): Merge-Review, PR-Align, PR-Gate and PR-Rework with its new plan generation, Finalize and the separate Describe and Merge-In rows in their current form, and verify failures sent back to Webster-Burler-Review.

## Background (audit, landed loom runs 2026-10-08 to 2026-10-22)

- PR-Rework ran in 8 of 130 runs, never twice, about $4.80 and 23 min each; 4 of 72 alignment verdicts rejected, all valid.
- Where those deviations entered: decision record 1, plan 3, code 3 (two written by the Webster-Review or Merge-Review fixer, which nothing reviews after it), not real 1.
- Merge-Review ran in 73% of runs, its fixer changed code in a third, about $0.96 per run.
- About 38 verify failures at Merge-In and Publish went back to Webster-Burler-Review.

## Open

- The Land block when its merge brings in a large or overlapping change, or its fix runs past a cap: escalate, or release the lock and (loom) go back to step 2.
  Orch suggestion: a cap on the fix's size and attempts under the lock, so a hard merge never holds the lock long.
- Orch suggestion: Describe writes the commit message before the lock is taken, and inside the Land block runs again only when the fix changed code, since the merge alone does not change what the task did; an agent turn under the lock holds every other landing.
- The step 2 fixer: twill or a solo strong agent, its attempt cap, and the size of merge change that triggers the perch round.
- Without a PR: `alignment_pr_approve`, the PR-closing comment and the GitHub record of a run.
