# Run tail: Merge-In with alignment, Code-Perch, a PR stub and one locked Land block, for loom and darn

`loom-tail-review` · Task · enhancement, loom, burler, shed, fabric, batten · paused · Merge-In

## Design (operator, 2026-10-23)

The run's tail after the code is written.

### Loom

1. **Webster** ends only when every test passes.
2. **Merge-In with alignment and a fixer**, without the parent's merge lock (today's Merge-In takes the lock and holds it until Finalize; that goes).
   It merges the parent in, resolves conflicts, runs the gated tests and fixes until they pass.
   The alignment agent's check becomes part of this fixer: it reads what the merge brought in against the decision record, fixes what follows from the record, and escalates a real conflict between decisions instead of choosing one.
   The fixer is one Opus agent, no twill: a large change is caught by this step together with the Code-Perch after it.
3. **Code-Perch**, a general perch over the task's code, in place of today's Webster perch, since the code is already designed in detail and tested before Webster ends.
   It runs after the merge on the bet that nothing large lands on the parent between this Merge-In and Finalize.
4. **Describe**, before the lock and before the PR block: it writes the landing's commit message, and a later PR's text, from the task's diff.
   A last merge-in under the lock is assumed too small to change the description, and a session handling a human PR review updates the description itself.
5. **PR block**, behind a config flag.
   For now it is a stub that does nothing; it grows into a human PR review (a PR for a human, and an interactive session that takes the human's feedback and fixes it) when one is needed.
   The run goes on while a human looks at a PR, as it does today.
6. **Finalize**, one producer for the whole locked operation (the name stays, since row names are resume identities):
   1. Take the parent's merge lock.
   2. Merge the parent in; on a conflict or a failing test, fix it.
      Run right, the parent brings nothing large here, so this is quick.
      When the merge left the tree as the one already verified, the tests are not run again (`pattern/PATTERN-verified-tree.md`).
      The `landing` gate is the one gate after Webster: Merge-In, Finalize and the darn writer run it, and since 2026-10-23 it holds every tier the gates run (`[0, 1, 2]`, the tmux tier included), so the `publish` gate goes.
   3. Land on the parent (squash with Describe's text as the commit message, naming the task's slug, push, board done).
      A run opens no PR unless the PR block's flag is on: the squash already lands locally, and the commit and the weft records are the run's history.
   4. Release the lock.

   The lock covers only this block, so the parent cannot move between the fix and the landing, and the lock is held as briefly as possible (as Millhouse does it).
   When the merge brings in a large change or the fix runs past a cap on its size or attempts, the block keeps the merge and fix on the branch, releases the lock and routes back through the recipe (loom: to step 2; darn: to Darn-Review), then lands again.
   A cap on these routes back ends in an ordinary stop to the orch, so a parent that keeps moving cannot loop the run.
   The route back is a recipe edge like today's Finalize-to-Merge-In, never a hand goto.

### Darn

A darn run is quick and narrow, and is not run beside work that can make it drift, so it has no Merge-In before its review: the writer (ending only when every test passes), Darn-Review as its Code-Perch, Describe, the PR block behind a darn flag of its own (separate from loom's, so it is a quick skip when off), and the same Finalize.
Loom has the step 2 Merge-In because a loom run can take hours.

### Failures are the recipe's own

Loom and darn fix their own test and merge failures inside these rows.
Batten's automatic retry of a run's first verify failure at Merge-In or Publish, which sends the run back to the failure record's target, is removed from batten.

### Gone

From today's tails (`contracts/recipes/loom-recipe.yaml`, `contracts/recipes/darn-recipe.yaml`): Merge-Review (its rows, its `merge_*` keys in `loom.yaml` and in the profiles), Publish's PR, PR-Align, PR-Gate, PR-Rework with its new plan generation, `lyx loom approve` and `lyx loom reject`, the PR-closing comment, `alignment_pr_approve`, and verify failures sent back to Webster-Burler-Review.
What reads them goes with them or is updated: `lyx report convergence`, and the orch role's PR-Gate section and its Merge-In/Publish verify-failure rule.

## Background (audit, landed loom runs 2026-10-08 to 2026-10-22)

- PR-Rework ran in 8 of 130 runs, never twice, about $4.80 and 23 min each; 4 of 72 alignment verdicts rejected, all valid.
- Where those deviations entered: decision record 1, plan 3, code 3 (two written by the Webster-Review or Merge-Review fixer, which nothing reviews after it), not real 1.
- Merge-Review ran in 73% of runs, its fixer changed code in a third, about $0.96 per run.
- About 38 verify failures at Merge-In and Publish went back to Webster-Burler-Review.

### Board tidy after landing

PR-Align's alignment agent today names the board entries a run supersedes, and the orch amends them.
That leaves loom, darn and batten: batten's job is one task, never other board entries.
The orch spawns a Sonnet agent at each landing that reports the notes the landing superseded (a hub-wide orch rule, not code in this task), and the orch's Go driver takes that job over once there is one.

## Open

- For the run's Discussion: the step 2 fixer's attempt cap, Finalize's caps on fix size, attempts and routes back, and what counts as a large merge.

## Landing this task

A run parked on a removed row when this lands is restarted, not migrated, so this task lands while no other run is in flight: the orch pauses its run before Merge-In with `lyx loom pause --before Merge-In`, since today's Merge-In takes the parent lock and holds it to Finalize, resumes it once no other run is in flight, and starts no run until it has landed.
