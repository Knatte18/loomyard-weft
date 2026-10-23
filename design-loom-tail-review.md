# Run tail: Merge-In with fixer before a Code-Perch, an optional PR block, and a locked re-merge at Finalize

`loom-tail-review` · Note · enhancement, loom, burler, shed, fabric, batten

## Design (operator, 2026-10-23)

The run's tail after the code is written, the same for loom and darn.

1. **Webster** (in darn, the writer) ends only when every test passes.
2. **Merge-In with alignment and a fixer.**
   It merges the parent in, resolves conflicts, runs the gated tests and fixes until they pass.
   The alignment agent's check becomes part of this fixer: it reads what the merge brought in against the decision record, fixes what follows from the record, and escalates a real conflict between decisions instead of choosing one.
   The fixer is thorough: a strong agent or a twill, and one perch round when the change the merge brought in is large.
3. **Code-Perch**, a general perch over the task's code, in place of today's Webster perch, since the code is already designed in detail and tested before Webster ends.
   It runs after the merge on the bet that nothing large lands on the parent between this Merge-In and Finalize.
4. **PR block**, only when a config flag is on: it opens a PR for a human, and an interactive session takes the human's feedback and fixes it.
   Its design waits until a human PR review is needed; with the flag off the run skips it.
5. **Finalize** takes the parent's merge lock, merges the parent in again, runs a light fixer (only small changes are expected here), lands on the parent (squash, push, board done) and releases the lock.
   The lock covers only this last merge, fix and landing, so the parent cannot move between the fix and the landing, and the lock is held as briefly as possible (as Millhouse does it).

Gone from today's tail (`contracts/recipes/loom-recipe.yaml`): Merge-Review, PR-Align, PR-Gate and PR-Rework with its new plan generation, and verify failures sent back to Webster-Burler-Review.

## Background (audit, landed loom runs 2026-10-08 to 2026-10-22)

- PR-Rework ran in 8 of 130 runs, never twice, about $4.80 and 23 min each; 4 of 72 alignment verdicts rejected, all valid.
- Where those deviations entered: decision record 1, plan 3, code 3 (two written by the Webster-Review or Merge-Review fixer, which nothing reviews after it), not real 1.
- Merge-Review ran in 73% of runs, its fixer changed code in a third, about $0.96 per run.
- About 38 verify failures at Merge-In and Publish went back to Webster-Burler-Review.

## Open

- Darn: whether its Merge-In also moves before its review, so Darn-Review becomes the Code-Perch.
- Finalize: what happens when its re-merge brings in a large or overlapping change: escalate, or release the lock and go back to step 2.
- The step 2 fixer: twill or a solo strong agent, its attempt cap, and the size of merge change that triggers the perch round.
- Without a PR: Describe's text as the squash commit message, `alignment_pr_approve`, the PR-closing comment and the GitHub record of a run.
