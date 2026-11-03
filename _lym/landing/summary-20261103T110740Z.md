# Tapestry's Discussion is a motif block, skipped for a discussed entry

Discussion is now defined once, in `contracts/recipes/motif-recipe.yaml`, and tapestry runs it through a `Motif` block row after Preflight.
A tapestry run whose board entry is flagged discussed passes over that block: the entry's stored decision record becomes the run's record and the run starts at Drift-Baseline.
A board task may also name `motif` to run Discussion alone: the run stores the approved record on the entry, flags it discussed and lands no code.
The `arras` recipe, which existed only to prove the block, is removed.

The task closes no issue.

## What changes

- **Tapestry's recipe.**
  `contracts/recipes/tapestry-recipe.yaml` drops its four Discussion rows for one block row, `Motif`, with `block: motif`, `block_gate: undiscussed` and `on_done: Drift-Baseline`.
  The rows and their comments move unchanged into `motif-recipe.yaml`.
  `TestTapestryMotifBlock` holds tapestry's expanded rows and top-level keys equal to `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`, a copy of the recipe before the move, config included.
  The only difference it admits is the block row and the Discussion rows' block path.
- **The `undiscussed` gate check.**
  It is a new row of `shedrecipe`'s `BlockGates` table and calls a new seam, `Env.SeedDiscussedRecord`.
  loomcli's seam reads the run's board entry through `Board.DecisionRecord`.
  For an entry flagged discussed, it writes the stored record to `_lym/discussion/decision-record.md`, writes a support log saying the record came from the board when none exists, and commits the discussion directory through `commitDiscussion`.
  The gate then passes over the block, and the skipped entry's reason names the discussed entry.
  Any other entry enters the block and runs Discussion as before.
- **`motif` runs alone.**
  It is in `shedrun.RecipeNames`, in the `lym shed` arming table, among the recipes Seed-Child and Run-Shed admit, and in the board's task-recipe refusal and `lym board upsert` help.
  It declares `contract: loom` and `profile_recipe: tapestry`, so a motif task takes tapestry's run-profile block.
  Its run is Discussion-Write and the Discussion-Review ring, and it ends on Discussion-Bouncer's Done.
  The approve seam already stores the record and sets the flag on that row.
- **Record-only loom contract.**
  `shedbuild`'s loom contract accepts a recipe with one record row and neither a build row nor a Finalize row, with no `verify_source`.
  A recipe with only one of a build row and a Finalize row is refused as before.
- **Block recipes' top-level keys.**
  `shedbuild.Expand` no longer refuses a block recipe that sets `contract`, `verify_source`, `amendment_target` or `profile_recipe`.
  It ignores them, as it already ignored `terminals`, so motif's keys serve only its own runs.
- **Board status of a run that lands nothing.**
  A finished run of a recipe with no Finalize row writes the `draft` status on its board entry (`finishedBoardStatus` in `internal/loomcli/wiring.go`).
  The entry leaves the run lock and dobby does not start it again.
  A recipe with a Finalize row still writes nothing when it finishes, since its landing marks the entry done.
- **Arbiter description.** The arbiter's recipe marker describes a recipe with no Finalize row as a task's Discussion that lands no code.
- **arras removed.**
  The recipe file, `recipes.ArrasRecipe` and `shedrun.RecipeArras` are deleted, along with every place that named arras: recipe keys, Seed-Child and Run-Shed, run reports, timing spans and tokencount.
  The timing table now counts tapestry's `Motif` row under discussion.
- **Docs.**
  Specs, stencils, the orch role, the board skill and package docs drop arras.
  They name motif where the task recipes are listed, and describe the record-only contract and the `undiscussed` check.

## Design decisions

- The pin against "before" is a testdata snapshot rather than a comparison built from the new recipe, so it checks the move itself.
  A deliberate change to tapestry's rows now also edits the snapshot; its header says so.
- motif sets no `verify_source`, so the loom contract gains a record-only form instead of motif carrying a `verify_source` it never reads.
- The finished motif run's entry goes to `draft`, not to `done` and not to an empty status.
  `done` would mean landed and would make the board refuse later record stores.
  An empty status would let dobby start the same task again at once.
  `draft` leaves the decision to the operator, who changes the recipe to tapestry and releases the task to build it.
- The seeding happens in the gate check because the decision record asks the check to read the stored record into `_lym/discussion`.
  The write is idempotent: a resume or a re-entry of the row writes the board's copy again.
  I judged this no departure from `PATTERN-shed-recipe-registry`'s "side action a Command row".
  The write is the block's own skip action, keyed by the check name in the one `BlockGates` table, not a row of its own.
- The support log is written only when missing, so a run that already holds one keeps it.
- The `Cartoon` name the board entry mentions was not in the code: the block row already carried `Motif`, and `cartoon` stays free for a later Plan block.
- motif runs neither Preflight nor Friction-Reflect, since the entry asks for Discussion-Write and its review and nothing else.
  A motif run's friction notes are therefore not reflected before its pair is torn down.
- Run reports and tokencount count finished tapestry and darn runs only.
  A motif run has no Finalize row, so it is never a full run there.
- Status panes number tapestry's rows one higher after Preflight, since the `Motif` row is an entry of its own.

## To check by hand

- No motif task has run end to end against a live agent.
  Check that batten's Land and Worktree-Teardown handle a child that finished with no Finalize.
  Land should pull, deploy and prune with nothing new, and teardown should keep the entry's `draft` status.
- Check that a dobby landing notice (`notify_on_land`) for a finished motif run reads sensibly, since the run reached Land without a landing commit.
- A goto back onto `Motif` in a run whose own approval already stored the record passes over the block and rewrites the working record with the board's copy.
  That copy also holds every entry `lym loom decision add` added, so nothing is lost; check that this bound is acceptable.
- The stencils and specs deployed under the hub's Bolt still name arras until the next deploy refreshes them.
- Peer task `drift-scope-in-run-settings` edits `contracts/orchrole/orch-role.md` and `docs/overview.md`, which this change also edits, so whichever lands second may need a textual merge.
