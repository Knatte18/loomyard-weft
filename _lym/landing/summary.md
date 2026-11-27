# Let a docs run's darn writer add and remove its own bench pairs

A docs bench that needs pairs of its own could not make them:
the fabric guard refuses every mutating `lym fabric` verb from a task session on the real hub,
so the writer had to ask the orch for each add and removal.

A darn writer on a run with profile `docs` now runs `lym fabric add` and `lym fabric remove` on its bench pairs itself,
and every run's Worktree-Teardown removes the bench pairs left behind.

## What the guard admits

- The one argument is a bench pair slug, `<task-slug>-b<N>`, where `N` is a positive number with no leading zero.
- The verb runs from the writer's task worktree or its records sibling.
- `add` takes neither `--parent` nor `--adopt-local`, so the pair forks from the task branch.
- `remove` has no flag restriction, so `--force` and `--remote` pass too.
  `--remote` irreversibly deletes the pair's weft branch on the weft remote,
  and also the landed task branch on the warp origin.
- The recipe (`darn`) and profile (`docs`) come from the run's identity record, `_lym/usage/run.json`, not from the board.
- A slug that a batten run directory on the prime names is refused, and the refusal says why.
  The guard reads no board, so a slug that only a board entry names passes.
- A detached push of a bench pair passes for the same writer under the same record and run directory checks.
- A darn writer's other refused `add` or `remove` calls now name the admitted form.

Nothing caps how many bench pairs a writer makes.

## What teardown does

Worktree-Teardown's push step first calls `pairteardown.RemoveBenchPairs`, with no prime lock held.
The removal goes by slug alone and reads no recipe or profile:
every run's teardown force-removes each `<task-slug>-b<N>` pair that no board entry or batten run directory names,
whoever made it.
For each such pair it:

1. ends the pair's registered processes, sparing the same roles as other teardowns;
2. commits any uncommitted files off the tip,
   then pushes an owner archive tag to origin for each resulting tip that origin lacks;
3. tears the pair down with force, leaving its branches on origin and settling no board claim.

- The processes end before the run-lock wait and the refusal probe, so a failed archive or removal leaves them ended.
  `pattern/PATTERN-pair-remote.md` records this as an exception to the order it lists.
- A half-removed bench pair is left with a warning that points at `lym fabric prune --apply`.
- A failure halts the row with the task pair kept, and a resume retries.
  The row's reason now reads "the push step failed", since the step does more than push.
- The bench pairs' archive tags are pushed even with git or pushes switched off.

Only Worktree-Teardown calls this removal.
Bench pairs left by a cancelled run are not removed by the cancel.

## To check by hand

The guard tests drive stand-in worktrees.
On a real hub, try `lym fabric add <task-slug>-b1` and its `remove` from a docs run's darn writer,
including the detached push that follows the add.

## Merge danger

**Door:** one-way
**Blast radius:** fabric
