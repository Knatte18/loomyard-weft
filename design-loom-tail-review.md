# Run tail: Merge-In with alignment, Code-Perch, a PR stub and one locked Land block, for loom and darn

`loom-tail-review` · Note · enhancement, loom, burler, shed, fabric, batten

## Design (operator, 2026-10-23)

The run's tail after the code is written.

### Loom

1. **Webster** ends only when every test passes.
2. **Merge-In with alignment and a fixer**, without the parent's merge lock.
   It merges the parent in, resolves conflicts, runs the gated tests and fixes until they pass.
   The alignment agent's check becomes part of this fixer: it reads what the merge brought in against the decision record, fixes what follows from the record, and escalates a real conflict between decisions instead of choosing one.
   The fixer is one Opus agent, no twill: a large change is caught by this step together with the Code-Perch after it.
3. **Code-Perch**, a general perch over the task's code, in place of today's Webster perch, since the code is already designed in detail and tested before Webster ends.
   It runs after the merge on the bet that nothing large lands on the parent between this Merge-In and the Land block.
4. **PR block**, behind a config flag.
   For now it is a stub that does nothing; it grows into a human PR review (a PR for a human, and an interactive session that takes the human's feedback and fixes it) when one is needed.
5. **Describe**, before the lock: it writes the landing's commit message from the task's diff, which a clean merge of the parent does not change.
6. **Land block**, one producer for the whole locked operation:
   1. Take the parent's merge lock.
   2. Merge the parent in; on a conflict or a failing test, fix it.
      Run right, the parent brings nothing large here, so this is quick.
   3. When the fix changed code, run Describe again, so the commit message matches what lands.
   4. Land on the parent (squash with Describe's text as the commit message, naming the task's slug, push, board done).
      A run opens no PR unless the PR block's flag is on: the squash already lands locally, and the commit and the weft records are the run's history.
   5. Release the lock.

   The lock covers only this block, so the parent cannot move between the fix and the landing, and the lock is held as briefly as possible (as Millhouse does it).
   When the merge brings in a large change or the fix runs past a cap on its size or attempts, the block keeps the merge and fix on the branch, releases the lock and routes back through the recipe (loom: to step 2; darn: to Darn-Review), then lands again.
   A cap on these routes back ends in an ordinary stop to the orch, so a parent that keeps moving cannot loop the run.
   The route back is a recipe edge like today's Finalize-to-Merge-In, never a hand goto.

### Darn

A darn run is quick and narrow, and is not run beside work that can make it drift, so it has no Merge-In before its review: the writer (ending only when every test passes), Darn-Review as its Code-Perch, the PR block behind a darn flag of its own (separate from loom's, so it is a quick skip when off), Describe, and the same Land block.
Loom has the step 2 Merge-In because a loom run can take hours.

### Gone

From today's tail (`contracts/recipes/loom-recipe.yaml`): Merge-Review, PR-Align, PR-Gate and PR-Rework with its new plan generation, Publish's PR, Finalize, the PR-closing comment, `alignment_pr_approve`, and verify failures sent back to Webster-Burler-Review.

## Background (audit, landed loom runs 2026-10-08 to 2026-10-22)

- PR-Rework ran in 8 of 130 runs, never twice, about $4.80 and 23 min each; 4 of 72 alignment verdicts rejected, all valid.
- Where those deviations entered: decision record 1, plan 3, code 3 (two written by the Webster-Review or Merge-Review fixer, which nothing reviews after it), not real 1.
- Merge-Review ran in 73% of runs, its fixer changed code in a third, about $0.96 per run.
- About 38 verify failures at Merge-In and Publish went back to Webster-Burler-Review.

## Open

- For the run's Discussion: the step 2 fixer's attempt cap, the Land block's caps on fix size, attempts and routes back, and what counts as a large merge.
