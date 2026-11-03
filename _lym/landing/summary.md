# lym beam create clears a done motif run under the project's slug

A motif run writes a task's decision record, and the orch may then turn the entry into a project to split it.
The project's beam runs under the same slug, so the motif run's done batten record made `lym beam create` refuse with "has already completed", and the operator had to run `lym batten cleanup --apply` from the prime first.
Now `lym beam create` clears that record itself and starts the pawl run.

## What changed

- **Child-recipe record** (`internal/shedrun/childrecipe.go`).
  `ChildRecipeFile`, `ChildRecipeRel`, `WriteChildRecipe` and `ReadChildRecipe` declare a durable `child-recipe` file in a batten run's own run directory, holding the recipe its task worktree's run was seeded with.
  The task worktree's own seed goes with the pair at teardown, so this is the only record that still names the child's recipe once a run is done.
- **Seed-Child writes it** (`internal/battencli/wire.go`).
  The `WriteSeed` seam writes the record in the prime's run directory after the child seed, and `battenRunCommitPaths` commits it with the status whenever it is on disk.
- **`battencli.ClearMotifRun`** (`internal/battencli/cleanup.go`).
  Under the slug's run lock and status lock, as `lym batten cleanup` judges a slug, it removes the records run directory and scratch of a run that is done and whose child-recipe record names `motif`, commits the removal on the records side and spawns the records push.
  A held run lock, a state other than done, another recipe or no record leaves the run alone and reports nothing cleared.
- **`lym beam create`** (`internal/beamcli/create.go`).
  Before the first pass, a done run whose pair is no longer on this machine goes to `ClearMotifRun`; the envelope reports a removal as `motif_run_cleared`.
  Any other done run stays, and the first pass still refuses it naming `lym batten cleanup --apply`.

## Design decisions

- **Clear at create, not remove at landing.**
  The record allowed either.
  Batten writes its final done status after its last row returns, so a motif run cannot delete its own record while it lands without a new post-done hook in the engine.
  Clearing at `lym beam create` keeps the change inside the one verb that needs the slug free.
- **A recorded recipe, not an inference.**
  Batten's done status carries no trace of its child's recipe: a motif run's Land row ends Done just as a code landing does.
  A recorded recipe is the only reading that tells a motif run from a landed code run whose slug a new project reuses, which must still refuse.
- **No backfill.**
  A run seeded before this change has no child-recipe record and keeps the old refusal and its `lym batten cleanup --apply` way forward.

## Tests

- `TestChildRecipe` pins the record's path, relative form and round trip.
- `TestBattenRunCommitPaths` gains the child-recipe record among the status commit's paths.
- `TestClearMotifRun_RemovesOnlyADoneMotifRun` covers a done motif run (removed, committed, pushed) and the runs left alone: done darn, done with no record, running motif, no status, held run lock.
- `TestCreate` gains rows for a cleared motif run before the start, a done run of another recipe left for the start to refuse, and a done run whose pair is still here not offered for clearing; `TestCreate_StepFailureNamesTheRerun` gains a clearing failure naming the re-run.
