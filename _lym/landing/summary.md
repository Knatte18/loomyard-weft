# A seed says only that a run starts at its start

A new loom run's status file used to name `Preflight` as its first producer.
`loomshed.Seed`, which the bootstrap of every loom recipe calls, wrote that name.
A recipe whose first row had another name stopped on its first step, because Shed's lookup found no such row.
After this change the seed names no producer: an empty `current_producer` means the run is at its start, and the recipe's first row is the start.

## What changes

- `loomshed.Seed` writes an empty `current_producer`.
  The rest of the seed stays the same: state running, an empty history, and the slug and parent in the product.
- `shedengine` adds `RowAt`, which turns an empty current producer into the name of the producer list's first row.
  A step's lookup reads the status through it.
  The step's pause check and every write it makes then use the row's real name, so the file never holds an empty `current_producer` after the first step.
  A `current_producer` that names a missing row still fails with the same error as before.
- `Routing.ProgressAt` reads an empty current producer the same way.
  Before the first step, `status` shows step 1, the first row, where it used to show step 0 with no name.
- The `Status.CurrentProducer` field doc, the `shedengine` package doc and `contracts/specs/loom-status-spec.md` describe the empty start.

The verbs that move a run need no change.
`goto` refuses a running run, and a fresh seed is running.
When `goto`'s reference row matches no row and no history entry, it already falls back to the first row.
`resume` passes the producer only to checks for awaiting runs and landing rows, and an empty value matches neither.

## Design decisions

- The start is the first row of the producer list, as the decision record's design states.
  The recipe's `entry` key is not used.
  Every recipe declares `entry` as its first row, but Shed does not receive `entry`.
- Shed resolves the empty producer when a step reads the status, not when the seed is written.
  The seed stays free of any recipe, so `tapestry`, `darn`, `pawl` and `sampler` all share it.
- Batten keeps its own status seeding, which names `Worktree-Create`.
  That seed is batten's alone and names a row in batten's own recipe, so the shared-seed defect does not apply to it.

## Tests

- `TestStep_EmptyCurrentProducerStartsAtTheFirstRow`: a step on an empty `current_producer` calls the first row, then records the next row and one history entry.
- `TestStep_EmptyCurrentProducerPausesBeforeTheFirstRow`: a `pause_before` on the first row still fires from an empty `current_producer`.
- `TestProgressAt` has a new row for an empty producer.
- `TestSeed_WritesExpectedStatus` now expects an empty `current_producer`.
- `TestSmokeBootstrapLifecycle` now checks that the status file left by the bootstrap has a state, instead of a named `current_producer`.
  That field is empty until the driver's first step and names a row after it.
- `TestRecipe_SeedAndResumeRowNamesExist` is removed.
  It checked that the tapestry recipe kept the row name the seed hard-coded, and the seed no longer hard-codes one.

## Open risks

- A recipe's `entry` and its first row are separate declarations.
  No check keeps them equal, and a recipe that sets them apart would start at its first row.
