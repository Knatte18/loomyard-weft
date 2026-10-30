# Pawl starts at a Preflight row, the row the shared seed names

A new beam's pawl run stopped at once with `current_producer "Preflight" ... names no producer in the list`.
`loomshed.Seed`, which the bootstrap shared by every inner recipe calls, always seeds `CurrentProducer` as `Preflight`, but the pawl recipe began at `Beam-Up` and had no such row.

## What changes

- `contracts/recipes/pawl-recipe.yaml` opens with the general `Preflight` row (engine `Preflight`, the `internal/preflightshed` producer loom and darn also run), and `entry` is now `Preflight`.
  The row hands to `Beam-Up` and carries no `on_stuck`, as in darn and loom.
- `internal/boardengine/runlock.go`: `Preflight` joins the hold-free producer rows, so a beam project whose run stands at `Preflight` holds nothing of its entry, as at `Beam-Up` and `Beam-Await`.
  The lift still applies only to a beam project; a loom or darn task at `Preflight` keeps its hold.
- Package docs that list the pawl recipe's rows or the hold-free rows name `Preflight`: `internal/loomrecipe`, `internal/loomcli`, `internal/boardengine`, `internal/dobbyengine`, and the slot comment in `internal/battencli/dobbyreads.go`.

## Design decisions

- One seed stays shared by every inner recipe; the recipe changes to match it, per the board entry's design.
- No retired-row mapping: no row is renamed or removed, so an in-flight pawl run past the seed keeps a valid status, and a run halted at the seed's `Preflight` resumes at the new row.
- The "producer list has changed" wording of the refusal the beam hit is left as it is; the mismatch no longer arises for pawl.

## Tests

- `internal/loomrecipe/pawlrecipe_test.go`: the row table and the routing entry now start at `Preflight`, of the general preflight producer's type; the hold-boundary test walks the new row list.
- `internal/loomcli/wiring_beam_test.go`: a pawl seed's routing rows start with `Preflight`.
- `internal/boardengine/runlock_test.go`: a `RunHolds` row for a beam project at `Preflight`.
