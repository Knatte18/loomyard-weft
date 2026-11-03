# Tapestry runs Discussion as a Motif block; motif runs alone; arras goes

Tapestry now defines Discussion once, in `motif-recipe.yaml`, and runs it through a block row named `Motif`.
When the board entry is flagged discussed, the block is passed over and the run starts at Drift-Baseline from the stored record, so Plan-Write and the reviews read it as they read a record the run approved.
An entry not flagged discussed runs Discussion as before.
`motif` is also a recipe a board task may name: it runs Discussion-Write and its review ring, stores the approved record on the board and flags the entry discussed, and lands no code.
The `arras` recipe, which was tapestry with nothing to bench against it, is removed, as is every place that named it.

## Design decisions

- **Block gate `undiscussed`** (`internal/shedrecipe/entries_block.go`).
  It calls `Env.SeedDiscussedRecord`; when that seeds a record it passes over the block with a stated reason, otherwise it holds.
  `loomcli` wires the seam to `seedDiscussedRecord`, which writes the stored record as the run's decision record, writes a support log (`loomshed.StoredRecordSupportLog`) when none exists, and commits the discussion directory.
  A read, write or commit failure fails the gate, and a resume runs it again.
- **A block recipe may carry run keys.**
  `shedbuild.Expand` no longer refuses a sub-recipe that sets `contract`, `verify_source`, `amendment_target` or `profile_recipe`; it ignores them, since the parent's stay the run's.
  `motif` needs this to run alone (`contract: loom`, `profile_recipe: tapestry`).
- **Loom contract has a record-only form.**
  A loom recipe with neither a build row nor a Finalize row needs only its record row and no `verify_source`.
- **A run with no Finalize finishes as `draft` on the board** (`finishedBoardStatus` in `internal/loomcli/wiring.go`), which releases the entry from its run lock without dobby starting it again.
  Recipes with a Finalize row write nothing, as before, since landing marks the entry done.
- **Landing a motif run only prunes the board.**
  `battenshed.LandDeps.ChildLanded` (wired in `internal/battencli/land.go` from the status history's Finalize outcome) lets `NewLand` skip the pull, deploy and landing notice for a child that landed no code.
- **Passed-approval test reads expanded rows.**
  `loomrecipe.PastDiscussionApproval` now finds the approval row inside the Motif block.
- **No run changes for tapestry.**
  `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml` pins tapestry's expanded rows, config included, to its rows before this change, and tests compare against it.
- **Removed `arras`** from `shedrun`, `loomrecipe` names and keys, Seed-Child, Run-Shed, run reports, timing spans, `profile_recipe` handling, the specs, docs and `tools/tokencount`.

## To check by hand

- Run a `motif` task end to end: the record lands on the board entry, the entry is flagged discussed and shows `draft`, and landing prunes it without a pull or deploy.
- Run a tapestry task on an entry already flagged discussed: Discussion is skipped and Plan-Write reads the stored record.
- The commit-status and seed wiring is pinned by tests for the finished status and for `seedDiscussedRecord`'s record write, board read and commit failures (`commitstatus_integration_test.go`, `discussionrecord_integration_test.go`).
  A review judged this coverage of the wiring still thin (scope, MEDIUM); check the pins are enough.
- `ChildLanded` treats only a Finalize with outcome Done as landed code, and a Finalize-less child only prunes the board.
  A review left this open as a design concern (MEDIUM); check that no recipe lands code without a Finalize row.
