# A shed block runs a whole sub-recipe inline as one producer, so recipes reuse one another

`shed-sub-run-block` · Task · enhancement, shed, loom · running · Discussion-Write

## Seen

A shed run starts at its first row, moves on by each row's outcome (`OnDone`, `OnStuck`) and ends done or stopped; a producer returns the same kind of outcome.
`battenshed.NewInnerRun` already treats a whole shed run as one producer: batten's Run-Shed row runs the task's loom run in the worktree batten made and goes on when it ends, and pawl does the same for a beam.
That block is batten-specific (a separate run with its own status file, driver, parent lock, owner check, notices, revive and binary re-step), and recipes cannot otherwise reuse one another: each is its own embedded YAML file in `loomrecipe`, and only the row entries in `shedrecipe` are shared.

## Idea (operator, 2026-11-03)

A general shed block that runs a whole sub-recipe as one producer of the parent run, so recipes are built from blocks instead of repeating rows.

- The block names a recipe; it is Done when the sub-run ends done and Stuck when it stops.
- Inline (operator, 2026-11-03): the sub-recipe's rows run inside the parent run, in the same process and status file, with their history entries marked with the block they belong to; no second run, driver or worktree, so nothing collides, and pause, goto and review budgets work as in any run.
- Each block has an entry gate (operator, 2026-11-03): a check with no agent, run before the block's first row, that either enters the block or passes over it whole, the block then counting as Done and the history recording it as passed over.
  Tapestry's gate on `cartoon` passes over it when the entry is already discussed, after reading the record from Bolt into `_lym/discussion` (note `discussed-task-split`).
  The recipe names the gate's check, from a set lym ships, so a gate is declared in the recipe file and not written as code per recipe.
- First uses: `cartoon` inside tapestry; later a part split. A sub-run in another worktree stays `NewInnerRun`'s kind.
- Today's tapestry stays unchanged while this is built: the first use is a new recipe beside it, `cartoon`'s block followed by tapestry's Plan and build rows, benched against tapestry on the same inputs, and tapestry moves to the block only once the bench shows no change in outcome or cost.

## Scope of this task (orch, 2026-11-03)

- In: the block and its entry gate in the shed engine and the recipe format, the set of gate checks lym ships with at least one that always enters, `cartoon` as a recipe of its own holding tapestry's Discussion rows, and a new recipe beside tapestry that runs `cartoon` through the block and then tapestry's Plan and build rows, so the block is proven on a real recipe; tapestry itself stays unchanged.
  Discussion proposes the new recipe's name, and the operator approves it.
- Out, as follow-up tasks: the gate check that reads a discussed entry's record from Bolt, which needs the record on the board from the core of note `discussed-task-split`; and the bench of the new recipe against tapestry.
- Open, for the run's Discussion to settle: how a goto, `lym loom status` and the progress count name a row inside a block; whether a block's review segments keep their own budgets; how recipe keys and profiles reach a block's rows.
