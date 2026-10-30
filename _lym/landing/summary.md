# Pawl recipe starts at Preflight, the row the shared seed names

A new beam's pawl run stopped at once with `current_producer "Preflight" ... names no producer in the list`.
`loomshed.Seed` sets `CurrentProducer` to `Preflight` for every inner recipe, and the pawl recipe had no such row: its entry was `Beam-Up`.
The pawl recipe now begins with a `Preflight` row, so the seed and the recipe agree.

## Changes

- `contracts/recipes/pawl-recipe.yaml`: a first row `Preflight`, engine `Preflight`, `on_done: Beam-Up`, and `entry: Preflight`.
  It carries no `on_stuck`, as in loom and darn.
  The header comment names the row and the new hold boundary.
- `internal/boardengine/runlock.go`: `Preflight` joins `holdFreeProducers`, since it stands before Parent-Align.
  A beam project whose run stands at `Preflight` takes members and scope edits, as at `Beam-Up`.
- Tests: `internal/loomrecipe/pawlrecipe_test.go` and `internal/loomcli/wiring_beam_test.go` pin the new first row; `internal/boardengine/runlock_test.go` adds a `BeamAtPreflight` case to `TestRunHolds`.
- Package docs and one comment (`internal/loomrecipe/doc.go`, `internal/loomcli/doc.go`, `internal/boardengine/doc.go`, `internal/dobbyengine/doc.go`, `internal/battencli/dobbyreads.go`) list `Preflight` among the pawl rows that hold nothing.

## Design decisions

- The row is the general producer in `internal/preflightshed`, reused through the existing registry entry; no new constructor.
- The seed stays shared and unchanged.
- The hold lift applies to beam projects only, because `RunHolds` lifts the hold for them alone; a loom or darn task at `Preflight` still holds its entry.
- No migration: no row is renamed or removed, so a run past its seed keeps a valid status file, and a run halted at the seed's `Preflight` resumes at the new row.
- The "producer list has changed" refusal wording is out of scope.

## To check by hand

- A freshly made beam pair passes the general Preflight checks, as a darn pair does; a pair left dirty by Worktree-Create now stops at `Preflight` instead of `Beam-Up`, and the stop names the failed check.
- Start a real beam under a hub and confirm it advances from `Preflight` to `Beam-Up`; no end-to-end run was done.
