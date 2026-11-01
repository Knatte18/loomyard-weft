# A seed says only that a run starts at its start, not which row

`loomshed.Seed` wrote `current_producer: "Preflight"` into every fresh status file, so each inner recipe's first row had to be named `Preflight`.
A new beam stopped at once because the pawl recipe has no such row.

The seed now writes an empty `current_producer`, which says only that the run starts at its start, the first row of the recipe's producer list.
One seed serves tapestry, darn and pawl, and can never name a row a recipe lacks.

## What changes

- `loomshed.Seed` leaves `CurrentProducer` empty.
- `shedengine.RowAt(producers, current)` is new: it returns `current`, or the first producer's name when `current` is empty (and `current` unchanged when there are no producers).
- A step resolves an empty `current_producer` to the first row at its lookup, so every write the step makes names the row itself.
  The resolved row is applied again after the status re-read that follows a gated first row's entry evaluation, because the file on disk still holds the empty value.
- `Routing.ProgressAt` resolves an empty `current` the same way, so progress before the first step reports the first row.
- The status spec (`contracts/specs/loom-status-spec.md`) and the comments in `loomshed`, `shedengine` and `loomengine` describe the empty start.
- `TestRecipe_SeedAndResumeRowNamesExist` is removed: it guarded the hard-coded `Preflight` name that no longer exists.
  `loomshed.NamePreflight` stays, since `interruptpolicy.go` still reads it.
- The bootstrap smoke test pins a seeded status by its state, since `current_producer` is empty until the first step.

## Design decisions

- Empty means "at the start" rather than a sentinel name, so no recipe has to reserve a row name.
- The resolution lives in `shedengine`, next to the lookup that already validates a named row. A named row that is not found is still a hard error that changes nothing on disk.
- A crash in the first row is indistinguishable from a fresh seed on disk, as it was before; `DetectCrashResume` still keys on a non-empty history.

## To check

- The task also asked to check the verbs that move a run (`goto`, resume) against an empty `current_producer`. New tests in `internal/shedengine/run_routing_test.go` cover a step from an empty start, a pause before the first row and a gated first row that halts; the `goto` path and a `lym loom` resume of a freshly seeded file are worth a read by hand.
- Any reader of `current_producer` outside `shedengine` that runs before the first step (status display, `loomengine` consumers) should tolerate an empty value; the diff changes only `ProgressAt`.
- The integration and tmux tiers were not run here.
