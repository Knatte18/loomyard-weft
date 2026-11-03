# Shed block rows run a whole sub-recipe inline; add motif and arras

A recipe can now reuse another recipe's rows through a block row, with no code per recipe.
A block row names a recipe (`block`) and a gate check (`block_gate`), and runs that recipe's rows inside the parent run, under the same process, status file and driver.
Pause, goto, resume and review budgets work on the inlined rows as in any run.
The block is proven on a real recipe: `motif` holds tapestry's Discussion rows, and the new `arras` runs `motif` through a block and then tapestry's rows from Drift-Baseline on.
Tapestry's recipe file is unchanged.

The task closes no issue.

## What changes

- **Recipe format.**
  A producer entry may be a block row carrying exactly `name`, `block`, `block_gate` and `on_done`.
  `shedbuild.Parse` refuses a block row that sets `engine`, `config`, `on_stuck`, `on_reroute`, `segment`, `max_bounces` or `entry_gate`, one missing `block` or `block_gate`, and a hand-written row with engine `Block`.
  A recipe may set a top-level `profile_recipe`, naming the recipe whose `profiles.yaml` block and default profile it takes.
- **Expansion.**
  `shedbuild.Expand` takes a parsed recipe and a told resolver from recipe name to bytes, and inlines each block's rows after its block row, innermost block first.
  Rows keep their own names and carry the path of enclosing blocks, joined by `/`.
  Every empty `on_done` of an inlined row is rewritten to the exit of its innermost block.
- **Run time.**
  `Block` is a producer in the `internal/shedrecipe` registry that returns Done at once.
  `shedengine.ProducerDef` gains a skip target, and a skipped row routes there when it is set and by `OnDone` otherwise, so every existing row skips as before.
  A gate that holds enters the block: the block row records Done and the run goes to the sub-recipe's entry.
  A gate that does not hold records a skipped entry with the reason and goes to the skip target, so no inlined row runs.
- **Gate checks.**
  `shedrecipe.BlockGates` is a closed table keyed by check name; it ships `always`.
  Build refuses a `block_gate` naming no check.
- **History and status.**
  `shedengine.HistoryEntry` gains an additive `block` field, omitted when empty.
  `lym loom status` shows a block row's Done as entered, and an inlined row's block beside its name in the row list and history.
- **Recipes.**
  `motif` is a block source only and is not in `shedrun.RecipeNames`.
  `arras` is seedable, so a board task may name it, and it is accepted and wired wherever `tapestry` is by name (batten Seed-Child and Run-Shed, run reports, timing spans, recipe keys).
  The board's `default_recipe` is unchanged.
- **Accessors.**
  `internal/loomrecipe` resolves every embedded recipe through one table, which is also the resolver given to `Expand`.
  The build, routing projection, row list, and row and engine lookups all read the expanded recipe.

## Design decisions

- A block has no `on_stuck`: a stop inside it halts the parent run at the inlined row that stopped, which is the same halt the row has in its own recipe's run.
- A block's end has no history entry; a reader finds it as the exit's first entry after the block row's latest Done.
- A review segment inside a block keeps its own name, so its budget and perch keys are the ones its own recipe reads; the Discussion-Review segment of `motif` reads the discussion perch keys as tapestry's does.
- `arras` sets `profile_recipe: tapestry`, so an `arras` task takes tapestry's profile block and default profile and an existing `profiles.yaml` serves it with no edit.
- Expand refuses an inlined row name or segment also used outside its block, so the same recipe used by two blocks in one parent is refused; accepted until a use needs it.
- Naming: `motif` is the Discussion-only recipe and produces a sketch, the decision record; `cartoon` is kept free for a later Plan block.
  Nothing in code is renamed to sketch.
- Tests pin `motif`'s rows equal to tapestry's Discussion rows and `arras`'s expanded rows equal to tapestry's plus the `Cartoon` row, config included, so the copies cannot drift.

## To check by hand

- A goto onto a block row enters the block without evaluating its gate, and a goto or re-launch onto an inlined row of a passed-over block runs that row with the gate never held; only a halted run can be moved, to a row `CheckGotoTarget` admits.
  Check that this bound is acceptable.
- Expansion rewrites `on_done` in nested blocks (`outer/inner`) by the innermost block's exit; only the tests cover a nested case, and no shipped recipe nests blocks.
- Peer task `decision-record-on-board` adds `approve_seam: discussion` to tapestry's Discussion-Bouncer; whichever lands second must carry the key into `motif`, which the config pin enforces.
- `arras` has not been run end to end against a live agent or benched against tapestry; both are follow-ups, as is the gate check that passes over `motif` for an already discussed entry.
- The verify gate failed once on `cmd/lym` with no failing package named, and a fix commit (keeping `lym board upsert`'s Long under the 2000-byte cap) cleared it; confirm the full gate is green.
